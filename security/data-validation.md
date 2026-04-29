## Data Validation

Data validation is the process of verifying that incoming data meets your expectations before you use it. Validation answers the question: "Is this data what I expect?" — checking type, format, range, and required fields.

Validation is different from sanitization (cleaning input) and escaping (securing output). All three are necessary, and each serves a different purpose.

### When to Validate

Validate data as early as possible — immediately after receiving it in your action file, before passing it to model classes or the database:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;

if (isset($params['submit'])) {
    $errors = [];

    // Validate required fields
    $name = trim($params['name'] ?? '');
    if (empty($name)) {
        $errors[] = $this->Lang('error_name_required');
    }

    // Validate date format
    $the_date = strtotime($params['the_date'] ?? '');
    if (!$the_date) {
        $errors[] = $this->Lang('error_invalid_date');
    }

    // Validate integer range
    $sort_order = (int) ($params['sort_order'] ?? 0);
    if ($sort_order < 0 || $sort_order > 9999) {
        $errors[] = $this->Lang('error_sort_order_range');
    }

    if (empty($errors)) {
        $holiday->name = $name;
        $holiday->the_date = $the_date;
        $holiday->save();
        $this->SetMessage($this->Lang('holiday_saved'));
        $this->RedirectToAdminTab();
    }

    // Errors exist — pass them to the template for display
    $tpl->assign('errors', $errors);
}
```

### Common Validation Techniques

#### Type checking

```
// Force integer
$id = (int) $params['hid'];

// Force boolean
$published = cms_to_bool($params['published']);

// Force float
$price = (float) $params['price'];
```

The `cms_to_bool()` utility function is provided by CMSMS and can interpret strings like `'yes'`, `'true'`, `'1'`, `'on'` as boolean true.

#### Required fields

```php
$name = trim($params['name'] ?? '');
if (empty($name)) {
    $errors[] = $this->Lang('error_name_required');
}
```

#### String length

```php
if (strlen($name) > 255) {
    $errors[] = $this->Lang('error_name_too_long');
}
```

#### Numeric range

```
$limit = (int) $params['limit'];
$limit = max(1, min($limit, 1000));
```

#### Pattern matching

```php
// Validate an email address
if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    $errors[] = $this->Lang('error_invalid_email');
}

// Validate a slug (letters, numbers, hyphens only)
if (!preg_match('/^[a-z0-9\-]+$/', $slug)) {
    $errors[] = $this->Lang('error_invalid_slug');
}
```

#### Allowed values (whitelist)

```php
$allowed_types = ['article', 'event', 'news'];
if (!in_array($params['type'], $allowed_types, true)) {
    $errors[] = $this->Lang('error_invalid_type');
}
```

### Frontend Parameter Registration

For frontend actions, CMSMS provides a first layer of validation through parameter registration. Parameters not registered in `InitializeFrontend()` are silently stripped from the request:

```php
public function InitializeFrontend()
{
    $this->SetParameterType('hid', CLEAN_INT);
    $this->SetParameterType('page', CLEAN_INT);
    $this->SetParameterType('detailpage', CLEAN_STRING);
}
```

This ensures that `hid` will always be an integer and `detailpage` will be a cleaned string. However, this is only a first pass — you should still validate the values in your action (e.g., check that the ID actually exists in the database).

### Validation in Model Classes

For reusable validation, put the logic in your model class:

```php
class HolidayItem
{
    public function is_valid()
    {
        if (!$this->name) return false;
        if (!$this->the_date) return false;
        return true;
    }

    public function save()
    {
        if (!$this->is_valid()) return false;
        // ... insert or update
    }
}
```

This way, validation runs regardless of which action triggers the save — admin edit, bulk import, API call, etc.

### Displaying Validation Errors

Pass errors to your Smarty template and display them to the user:

```smarty
{if !empty($errors)}
<div class="pageerror">
    <ul>
    {foreach $errors as $error}
        <li>{$error}</li>
    {/foreach}
    </ul>
</div>
{/if}
```

### Next Steps

Continue to [Nonces (CSRF Protection)](/modules/security/nonces/) to learn how to protect your forms against cross-site request forgery.
