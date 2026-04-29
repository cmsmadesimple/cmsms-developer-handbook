## Using Lang Strings

The `Lang()` method is how you access translated strings throughout your module — in PHP action files, module class methods, and Smarty templates.

### In PHP

```php
// Simple string lookup
$message = $this->Lang('holiday_saved');

// With substitution parameters (uses sprintf internally)
$message = $this->Lang('items_found', $count);
// lang file: $lang['items_found'] = '%d holidays found';

// Multiple parameters
$message = $this->Lang('welcome_user', $username, $role);
// lang file: $lang['welcome_user'] = 'Welcome %s, you are logged in as %s';
```

### In Smarty Templates

```html
{* Simple string *}
&lt;h3&gt;{$mod->Lang('add_holiday')}&lt;/h3&gt;

{* As a button label *}
&lt;input type="submit" name="{$actionid}submit" value="{$mod->Lang('submit')}" /&gt;

{* In a link *}
&lt;a href="{$edit_url}" title="{$mod->Lang('edit')}"&gt;{admin_icon icon='edit.gif'}&lt;/a&gt;

{* With parameters *}
&lt;p&gt;{$mod->Lang('items_found', $total)}&lt;/p&gt;
```

### Common Usage Patterns

```php
// Module class methods
public function GetFriendlyName() { return $this->Lang('friendlyname'); }
public function GetAdminDescription() { return $this->Lang('admindescription'); }
public function UninstallPreMessage() { return $this->Lang('ask_uninstall'); }

// Admin action messages
$this->SetMessage($this->Lang('holiday_saved'));
$this->SetError($this->Lang('error_saving'));

// Event descriptions
public function GetEventDescription($eventname)
{
    return $this->Lang('event_desc_' . $eventname);
}

// Parameter help
public function InitializeAdmin()
{
    $this->CreateParameter('hid', null, $this->Lang('param_hid'));
}
```

### The Translation Center

The CMSMS Translation Center is an online tool that allows community volunteers to translate module strings into different languages. To enable translations for your module:

1. **Register your module on the CMSMS Forge** (dev.cmsmadesimple.org).
2. **Contact the translation team lead** with your project name, Forge unix name, and the public URL to your `lang/en_US.php` file.
3. **Pull translations before releases.** Translations are stored in the Translation Center's SVN repository. Configure your source control to pull from `lang/ext/` before packaging a release.

For SVN-based projects:

```
cd modules/Holidays/
svn propedit svn:externals lang
# Add: ext http://svn.cmsmadesimple.org/svn/translatecenter/modules/Holidays/lang/ext
```

For Git-based projects, use a subtree or manually download the translations before release.

### Best Practices

- Use Lang() for every user-facing string — never hardcode text in PHP or templates.
- Use descriptive key names: `error_name_required` is better than `err1`.
- Keep the `en_US.php` file as the single source of truth for all strings.
- Use `sprintf`-style placeholders (`%s`, `%d`) for dynamic values instead of concatenation.
- Permission names cannot be translated — keep them in English.

### Next Steps

This completes the Internationalization chapter. Continue to [The CMSMS Forge](/modules/cmsms-forge/) to learn how to package and distribute your module.
