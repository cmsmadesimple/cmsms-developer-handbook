## Securing (Sanitizing) Input

Input sanitization is the process of cleaning data before you use it — removing or neutralizing anything that shouldn't be there. While validation checks whether data is correct, sanitization makes data safe.

### The Input Flow

Every piece of user input should pass through this pipeline:

1. **Register** — declare expected parameters and their types (frontend only).
2. **Sanitize** — clean the data (trim, cast, strip).
3. **Validate** — check that the cleaned data meets your requirements.
4. **Use** — pass the clean, validated data to your model or database.

### Frontend Parameter Registration

CMSMS provides automatic sanitization for frontend parameters through `SetParameterType()`. This is the first line of defense — unregistered parameters are stripped entirely:

```php
public function InitializeFrontend()
{
    $this->SetParameterType('hid', CLEAN_INT);
    $this->SetParameterType('page', CLEAN_INT);
    $this->SetParameterType('search', CLEAN_STRING);
    $this->SetParameterType('detailpage', CLEAN_STRING);
}
```

#### Cleaning constants

| Constant | Effect |
| --- | --- |
| `CLEAN_INT` | Casts to integer |
| `CLEAN_FLOAT` | Casts to float |
| `CLEAN_STRING` | Strips tags and trims whitespace |
| `CLEAN_NONE` | No cleaning — use with caution |

> **Note:** Admin action parameters are not filtered through `SetParameterType()`. You must sanitize them manually in your action files.

### Manual Sanitization in Actions

In admin actions (and as a second pass in frontend actions), sanitize input explicitly:

#### Strings

```
// Trim whitespace
$name = trim($params['name'] ?? '');

// Strip HTML tags from plain-text fields
$name = strip_tags(trim($params['name'] ?? ''));
```

#### Integers

```
// Cast to integer
$id = (int) ($params['hid'] ?? 0);

// Ensure positive
$page = max(1, (int) ($params['page'] ?? 1));
```

#### Booleans

```
// Use CMSMS utility — handles 'yes', 'true', '1', 'on', etc.
$published = cms_to_bool($params['published'] ?? false);
```

#### Dates

```
// Convert to unix timestamp
$the_date = strtotime($params['the_date'] ?? '');
if (!$the_date) {
    // Invalid date — handle the error
}
```

#### Rich text (WYSIWYG content)

Fields edited with a WYSIWYG editor intentionally contain HTML. Do not strip all tags, but do sanitize to remove dangerous elements:

```
// Allow safe HTML, remove scripts and event handlers
$description = $params['description'] ?? '';

// At minimum, remove script tags
$description = preg_replace('/<script\b[^>]*>(.*?)<\/script>/is', '', $description);
```
> **Note:** WYSIWYG editors like MicroTiny or TinyMCE perform their own sanitization, but you should not rely on client-side filtering alone. Always sanitize on the server.

### Using CMSMSExt for Input Sanitization

The [CMSMSExt](https://dev.cmsmadesimple.org/projects/cmsmsext) module provides the `xt_param` class — a type-safe parameter extraction utility that replaces all of the manual sanitization above with clean one-liners. Each method handles defaults, null-safety, and appropriate filtering internally.

```
// Integers
$id = \xt_param::get_int($params, 'hid', 0);
$page = \xt_param::get_int($params, 'page', 1);

// Strings (strips tags, filters special chars)
$name = \xt_param::get_string($params, 'name', '');

// Booleans (handles 'yes', 'true', '1', 'on', etc.)
$published = \xt_param::get_bool($params, 'published', false);

// Floats
$price = \xt_param::get_float($params, 'price', 0.0);

// Dates (returns unix timestamp or default)
$the_date = \xt_param::get_date($params, 'the_date');

// Rich text / HTML (sanitized via htmLawed)
$description = \xt_param::get_html($params, 'description', '');

// Arrays of strings
$tags = \xt_param::get_string_array($params, 'tags', []);
```

This is the recommended approach when your module already depends on CMSMSExt. It eliminates repetitive sanitization boilerplate and ensures consistent filtering across your codebase.

### SQL Injection Prevention

The most critical form of input sanitization is preventing SQL injection. Always use parameterized queries:

```sql
// SAFE — parameterized query
$db = \cms_utils::get_db();
$sql = 'SELECT * FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE id = ?';
$row = $db->GetRow($sql, [(int) $params['hid']]);

// SAFE — multiple parameters
$sql = 'INSERT INTO ' . CMS_DB_PREFIX . 'mod_holidays (name, the_date) VALUES (?, ?)';
$db->Execute($sql, [$name, $the_date]);

// DANGEROUS — never do this
$sql = "SELECT * FROM " . CMS_DB_PREFIX . "mod_holidays WHERE id = " . $params['hid'];
$row = $db->GetRow($sql);
```

The `?` placeholders are replaced by the database abstraction layer, which properly quotes and escapes the values based on their data type.

### File Upload Sanitization

If your module accepts file uploads:

- Check the file extension against a whitelist of allowed types.
- Check the MIME type — do not trust the extension alone.
- Limit file size.
- Generate a safe filename — never use the original filename directly.
- Store uploads outside the module directory (use the CMSMS uploads path).

```php
$allowed_extensions = ['jpg', 'jpeg', 'png', 'gif', 'pdf'];
$ext = strtolower(pathinfo($_FILES['upload']['name'], PATHINFO_EXTENSION));

if (!in_array($ext, $allowed_extensions, true)) {
    $this->SetError($this->Lang('error_invalid_file_type'));
    $this->RedirectToAdminTab();
}

// Generate a safe filename
$safe_name = uniqid('holiday_') . '.' . $ext;
$config = \cms_config::get_instance();
$dest = $config['uploads_path'] . DIRECTORY_SEPARATOR . 'holidays' . DIRECTORY_SEPARATOR . $safe_name;

move_uploaded_file($_FILES['upload']['tmp_name'], $dest);
```

### Summary: Sanitize by Data Type

| Data type | Manual | CMSMSExt |
| --- | --- | --- |
| Integer | `(int)` cast | `xt_param::get_int()` |
| Float | `(float)` cast | `xt_param::get_float()` |
| Boolean | `cms_to_bool()` | `xt_param::get_bool()` |
| Plain text string | `trim()`, `strip_tags()` | `xt_param::get_string()` |
| Rich text (HTML) | Remove `<script>` tags and event handlers | `xt_param::get_html()` |
| Date | `strtotime()` with validation | `xt_param::get_date()` |
| File upload | Whitelist extension, check MIME, generate safe filename | — |
| SQL values | Parameterized queries (`?` placeholders) | — |

### Next Steps

This completes the Module Security chapter. Continue to [Events](/events) to learn how modules communicate with each other and the CMSMS core through the event system.
