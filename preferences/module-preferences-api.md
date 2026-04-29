## Module Preferences API

Module preferences are key-value pairs scoped to your module. They are the most common way to store module settings — items per page, default templates, feature toggles, API keys, and other configuration values.

CMSMS automatically namespaces module preferences so they cannot collide with preferences from other modules.

### Setting Preferences

```php
// Set a preference
$this->SetPreference('items_per_page', 20);
$this->SetPreference('default_template', 'summary');
$this->SetPreference('enable_comments', 'true');
```

Values are stored as strings. If you need to store integers or booleans, cast them when reading.

### Getting Preferences

```php
// Get a preference with a default fallback
$per_page = (int) $this->GetPreference('items_per_page', 20);
$template = $this->GetPreference('default_template', 'summary');
$comments = cms_to_bool($this->GetPreference('enable_comments', 'false'));
```

The second argument is the default value returned if the preference does not exist.

### Removing Preferences

```php
// Remove a specific preference
$this->RemovePreference('items_per_page');

// Remove ALL preferences for this module (no argument)
$this->RemovePreference();
```

Calling `RemovePreference()` with no argument removes every preference belonging to your module. Use this in `method.uninstall.php`.

### Listing Preferences by Prefix

```php
// List all preference names that start with 'email_'
$prefs = $this->ListPreferencesByPrefix('email_');
// Returns: ['email_from', 'email_subject', 'email_template'] or null
```

This is useful when your module stores a dynamic set of preferences with a common prefix.

### Setting Defaults on Install

Set default preference values in `method.install.php`:

```php
// method.install.php
if (!defined('CMS_VERSION')) exit;

$this->SetPreference('items_per_page', 20);
$this->SetPreference('default_template', 'summary');
$this->SetPreference('enable_comments', 'false');
```

### Cleaning Up on Uninstall

Remove all module preferences in `method.uninstall.php`:

```php
// method.uninstall.php
if (!defined('CMS_VERSION')) exit;

$this->RemovePreference();
```
> **Note:** If your module returns `true` from `AllowUninstallCleanup()` (the default), CMSMS will automatically remove all module preferences on uninstall. You only need to call `RemovePreference()` explicitly if you override that method to return `false`.

### Using Preferences in a Settings Form

A typical pattern for a module settings tab:

```php
// action.defaultadmin.php (settings tab processing)
if (isset($params['save_settings'])) {
    $this->SetPreference('items_per_page', (int) $params['items_per_page']);
    $this->SetPreference('default_template', trim($params['default_template']));
    $this->SetPreference('enable_comments', isset($params['enable_comments']) ? 'true' : 'false');
    $this->SetMessage($this->Lang('settings_saved'));
    $this->RedirectToAdminTab('settings');
}

// Pass current values to the template
$tpl->assign('items_per_page', (int) $this->GetPreference('items_per_page', 20));
$tpl->assign('default_template', $this->GetPreference('default_template', 'summary'));
$tpl->assign('enable_comments', cms_to_bool($this->GetPreference('enable_comments', 'false')));
```

### API Reference

| Method | Description |
| --- | --- |
| `$this->SetPreference($key, $value)` | Set a module preference. Creates it if it doesn't exist. |
| `$this->GetPreference($key, $default)` | Get a module preference. Returns `$default` if not found. |
| `$this->RemovePreference($key)` | Remove a specific preference. Call with no argument to remove all. |
| `$this->ListPreferencesByPrefix($prefix)` | List preference names matching a prefix. Returns array or null. |

### Next Steps

Continue to [Site and User Preferences](/modules/preferences/site-preferences/) to learn about site-wide and per-user preference storage.
