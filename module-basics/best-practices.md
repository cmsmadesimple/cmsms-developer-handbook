## Best Practices

Following these conventions will make your module more secure, maintainable, and compatible with the CMSMS ecosystem. Many of these patterns are enforced by tools like [Module Check](/module-check) and the [CMSMS Scanner](/cmsms-scanner).

### Module Naming

Your module name is used as the folder name, the class name, and the identifier throughout CMSMS. Choose it carefully:

- **PascalCase** — e.g., `Holidays`, `MyGallery`, `EventCalendar`. Not `my_gallery` or `eventcalendar`.
- **Folder must match class name exactly** — if your class is `EventCalendar`, the folder must be `modules/EventCalendar/`.
- **Be specific** — avoid generic names like `Utils`, `Helper`, `Tools`, or `Manager` that could collide with other modules.
- **Don't conflict with core** — avoid names already used by CMSMS core modules (e.g., `Search`, `News`, `FileManager`).
- **Match your Forge project name** — when registering on the Forge, the "Full Name" should match your module's friendly name and the "Unix Name" should be the lowercase version of your module folder name (e.g., `holidays` for `Holidays`). The Unix Name is used in your repository URL and cannot be changed later, so choose carefully.

### Avoid Naming Collisions

The CMSMS `modules/` directory can contain dozens of modules from different authors. To avoid collisions, prefix your module-specific names with your module name or a unique short prefix:

- **Permissions** — e.g., `manage_holidays` not `manage`.
- **Database tables** — e.g., `mod_holidays` not `mod_items`.
- **Preference keys** — e.g., `holidays_items_per_page` not `items_per_page`.
- **Event names** — e.g., `HolidayItemAdded` not `ItemAdded`.
- **Smarty variable names** — e.g., `holiday_list` not `items`.
- **Library classes** — use the module namespace or prefix class names to avoid colliding with classes from other modules in the `lib/` autoloader.

### Don't Reinvent the Wheel

Before writing new functionality, check whether CMSMS core or an existing module already provides it:

- Need to send email? Use the **CMSMailer** module rather than calling `mail()` directly.
- Need a CAPTCHA? Use the **Captcha** module's API.
- Need search indexing? Integrate with the **Search** module.
- Need file picking? Use the **FileManager** module.
- Need WYSIWYG editing? Use the `{cms_textarea}` Smarty plugin — it automatically integrates with whatever WYSIWYG module is installed.

The CMSMS API also provides utility functions for common tasks like database access, URL generation, cookie handling, and remote HTTP requests. Check the [API documentation](https://docs.cmsmadesimple.org/) before writing your own.

### Security Checks in Every File

Every PHP file in your module should begin with a check that prevents direct browser access:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;
```

This ensures the file can only be executed within the CMSMS application context.

### Permission Checks in Admin Actions

Every admin action file should verify that the current user has the required permission before doing anything:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(MyModule::MANAGE_PERM)) return;
```

This protects against users who somehow obtain a direct URL to an admin action without having the proper access rights.

### Use Class Constants for Permission Names

Define permission names as class constants to avoid typos and make them easy to reference throughout your module:

```
class MyModule extends CMSModule
{
    const MANAGE_PERM = 'manage_mymodule';
}
```

### Use Language Strings

Never hardcode user-facing text in your PHP or template files. Always use the `Lang()` method so strings can be translated:

```php
// Good
$this->SetMessage($this->Lang('item_saved'));

// Bad
$this->SetMessage('The item has been saved');
```

### Use Parameterized Queries

Always use `?` placeholders in SQL statements and pass values as an array. Never concatenate user input into SQL strings:

```sql
// Good — parameterized query
$db->Execute('SELECT * FROM ' . CMS_DB_PREFIX . 'mod_items WHERE id = ?', [$id]);

// Bad — SQL injection risk
$db->Execute('SELECT * FROM ' . CMS_DB_PREFIX . "mod_items WHERE id = $id");
```

### Register Frontend Parameters

For security, all parameters passed to frontend actions must be registered in `InitializeFrontend()` with their expected type. Unregistered parameters are silently stripped:

```php
public function InitializeFrontend()
{
    $this->SetParameterType('hid', CLEAN_INT);
    $this->SetParameterType('page', CLEAN_INT);
    $this->SetParameterType('detailpage', CLEAN_STRING);
}
```

Available cleaning constants include `CLEAN_INT`, `CLEAN_STRING`, `CLEAN_FLOAT`, and `CLEAN_NONE`. Parameters only need to be registered once for all frontend actions.

### Separate Concerns (MVC)

CMSMS supports the Model/View/Controller pattern:

- **Actions** (controllers) — handle logic, form processing, and data retrieval. No HTML output.
- **Templates** (views) — handle all HTML rendering. No business logic.
- **Model classes** — handle data access and validation. Placed in `lib/`.

Keep your action files clean and functional. Do not mix HTML output into your PHP controllers, and do not put business logic into your Smarty templates.

### Create New Smarty Scopes

When rendering templates from an action, create a new Smarty template scope to avoid accidentally overwriting variables from other actions or the parent template:

```php
$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource('mytemplate.tpl'), null, null, $smarty
);
$tpl->assign('items', $items);
$tpl->display();
```

### Post-Redirect-Get (PRG) Pattern

Admin actions that process form submissions should follow the PRG pattern: process the form data, set a success/error message, then redirect. This prevents duplicate submissions when the user refreshes the page:

```php
if (isset($params['submit'])) {
    // Process form data...
    $item->save();
    $this->SetMessage($this->Lang('item_saved'));
    $this->RedirectToAdminTab();
}

if (isset($params['cancel'])) {
    $this->RedirectToAdminTab();
}
```

### Prefix Input Field Names

Every form input field in a module template must be prefixed with `{$actionid}`. This is how CMSMS routes form data back to the correct action, especially on the frontend where the same module can be called multiple times on one page:

```html
&lt;input type="text" name="{$actionid}name" value="{$item->name}" /&gt;
```

The `{form_start}` Smarty plugin handles the form tag and hidden fields automatically. Parameters like `hid=$item->id` passed to `{form_start}` are automatically prefixed.

### Lazy Loading

Modules can opt into lazy loading to avoid being loaded into memory on every request. This is especially useful for admin-only modules:

```
public function LazyLoadAdmin() { return true; }
```

However, lazy loading has restrictions. Your module cannot be lazy loaded in certain scenarios:

| Scenario | Frontend lazy load | Admin lazy load |
| --- | --- | --- |
| Registers module name dynamically (RegisterModulePlugin) | No | Yes |
| Registers routes dynamically | No | Yes |
| Registers additional Smarty plugins | No | No |
| Shares APIs with other modules | Yes\* | Yes\* |
| Registers content types | No | No |
| Registers module block types | Yes | Yes |

\* Other modules may need to explicitly request your module instance or list it as a dependency.

### Only Load What You Need

Beyond lazy loading the module itself, avoid loading heavy libraries or performing expensive operations until they are actually needed:

```
// Good — only load the PDF library when generating a PDF
if (isset($params['export_pdf'])) {
    require_once __DIR__ . '/lib/vendor/pdf_library.php';
    // generate PDF...
}

// Bad — loading it on every request
require_once __DIR__ . '/lib/vendor/pdf_library.php';
```

### Use OOP for Helper Code

While CMSMS action files are procedural by nature, your supporting code in `lib/` should use object-oriented patterns. This makes code easier to reuse, test, and maintain:

- Create model classes for your data entities (e.g., `class.HolidayItem.php`).
- Create query/repository classes for database access (e.g., `class.HolidayQuery.php`).
- Keep action files thin — delegate logic to your classes.

### No Closing PHP Tags

Do not include a closing `<?>` tag at the end of PHP files. This prevents accidental whitespace or newlines after the tag from being sent as output, which can cause "headers already sent" errors.

### Table Naming

Always prefix your database table names with `CMS_DB_PREFIX` followed by `mod_` and your module's short name:

```
CMS_DB_PREFIX . 'mod_holidays'
```

This avoids collisions with core tables and other modules.

### Error Handling

Production modules should include proper error handling. Consider throwing exceptions in your model classes and using try/catch blocks in your actions:

```php
try {
    $item = MyItem::load_by_id((int) $params['id']);
    if (!$item) throw new \CmsException('Item not found');
    $item->delete();
    $this->SetMessage($this->Lang('item_deleted'));
} catch (\Exception $e) {
    $this->SetError($e->getMessage());
}
$this->RedirectToAdminTab();
```

### Working with Other Modules

When interacting with another module's code:

- **Do not** write directly to another module's database tables, files, or cache. Always use its public, documented API methods.
- **Do not** use undocumented or internal methods — they may change or disappear without notice.
- **Do** check that the other module is available before calling it:

```
$captcha = \cms_utils::get_module('Captcha');
if ($captcha) {
    $captcha->SomePublicMethod();
}
```

If your module absolutely requires another module to function, add it to your `GetDependencies()` return value. If it is optional (like Search integration), check for its existence at runtime without creating a hard dependency.
