## Custom Permissions

Most modules need at least one custom permission to control who can access their admin panel. Some modules define multiple permissions for different access levels.

### Creating Permissions

Create permissions in `method.install.php` and define the names as class constants:

```php
// Holidays.module.php
class Holidays extends CMSModule
{
    const MANAGE_PERM = 'manage_holidays';
    const VIEW_PERM   = 'view_holidays';
}

// method.install.php
$this->CreatePermission(Holidays::MANAGE_PERM, 'Manage Holidays');
$this->CreatePermission(Holidays::VIEW_PERM, 'View Holidays');
```

The first argument is the internal permission name (stored in the database). The second is the human-readable label displayed in the admin panel under Users & Groups.

> **Note:** CMSMS does not support localized permission names. The human-readable string should be in English.

### Checking Permissions

```php
// In VisibleToAdminUser() — controls navigation visibility
public function VisibleToAdminUser()
{
    return $this->CheckPermission(self::MANAGE_PERM)
        || $this->CheckPermission(self::VIEW_PERM);
}

// In admin actions — controls access
// action.defaultadmin.php (read-only view)
if (!$this->CheckPermission(Holidays::VIEW_PERM)
    && !$this->CheckPermission(Holidays::MANAGE_PERM)) return;

// action.edit_holiday.php (write access)
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;

// action.delete_holiday.php (write access)
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;
```

### Conditional UI Based on Permissions

Show or hide UI elements in templates based on the user's permissions:

```smarty
{* In your admin template *}
{if $mod->CheckPermission('manage_holidays')}
  &lt;div class="pageoptions"&gt;
    &lt;a href="{cms_action_url action=edit_holiday}"&gt;
      {admin_icon icon='newobject.gif'} {$mod->Lang('add_holiday')}
    &lt;/a&gt;
  &lt;/div&gt;
{/if}

{foreach $holidays as $holiday}
  &lt;tr&gt;
    &lt;td&gt;{$holiday->name|escape}&lt;/td&gt;
    {if $mod->CheckPermission('manage_holidays')}
      &lt;td&gt;&lt;a href="{cms_action_url action=edit_holiday hid=$holiday->id}"&gt;
            {admin_icon icon='edit.gif'}&lt;/a&gt;&lt;/td&gt;
      &lt;td&gt;&lt;a href="{cms_action_url action=delete_holiday hid=$holiday->id}"&gt;
            {admin_icon icon='delete.gif'}&lt;/a&gt;&lt;/td&gt;
    {/if}
  &lt;/tr&gt;
{/foreach}
```

### Removing Permissions

Remove all custom permissions in `method.uninstall.php`:

```php
$this->RemovePermission(Holidays::MANAGE_PERM);
$this->RemovePermission(Holidays::VIEW_PERM);
```

### Adding Permissions in Upgrades

```php
// method.upgrade.php
if (version_compare($oldversion, '1.2', '&lt;')) {
    $this->CreatePermission(Holidays::VIEW_PERM, 'View Holidays');
}
```

### Permission Design Patterns

| Pattern | Permissions | Use case |
| --- | --- | --- |
| Single permission | `manage_mymodule` | Simple modules where all admin users have full access |
| View / Manage | `view_mymodule`, `manage_mymodule` | Modules where some users can view but not edit |
| Granular | `add_items`, `edit_items`, `delete_items`, `manage_settings` | Complex modules with fine-grained access control |

For most modules, a single `manage_modulename` permission is sufficient. Add more only when you have a clear need for different access levels.

### Next Steps

This completes the Users and Permissions chapter. Continue to [Internationalization](/modules/internationalization/) to learn how to make your module translatable.
