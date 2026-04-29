## Checking Permissions

CMSMS has a built-in permission system that controls what admin users can do. Your module should create its own permissions during installation and check them in every admin action.

### Why Permission Checks Matter

The admin navigation only shows modules to users who pass the `VisibleToAdminUser()` check. But this is just a UI convenience — it does not prevent a user from accessing an action directly via URL. Without explicit permission checks in each action file, any authenticated admin user could manipulate your module's data.

### Creating Permissions

Create permissions in your `method.install.php` and define the name as a class constant:

```php
// In your module class
class Holidays extends CMSModule
{
    const MANAGE_PERM = 'manage_holidays';
}

// In method.install.php
$this->CreatePermission(Holidays::MANAGE_PERM, 'Manage Holidays');
```

After installation, site administrators can assign this permission to specific admin users or groups via Users & Groups in the admin panel.

> **Note:** Admin users who are members of the "admin" group automatically have all permissions without needing them explicitly assigned.

### Checking Permissions in VisibleToAdminUser()

This method controls whether your module appears in the admin navigation for the current user:

```php
public function VisibleToAdminUser()
{
    return $this->CheckPermission(self::MANAGE_PERM);
}
```

This is necessary but not sufficient — it only hides the navigation link.

### Checking Permissions in Every Admin Action

Every admin action file must independently verify permissions at the top, before any processing:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;

// ... rest of the action
```

This pattern must appear in every `action.*.php` file that performs admin operations — including edit, delete, and settings actions.

### Multiple Permissions

For modules with different access levels, create multiple permissions:

```php
// In your module class
const MANAGE_PERM = 'manage_holidays';
const VIEW_PERM   = 'view_holidays';

// In method.install.php
$this->CreatePermission(Holidays::MANAGE_PERM, 'Manage Holidays');
$this->CreatePermission(Holidays::VIEW_PERM, 'View Holidays');
```

Then check the appropriate permission in each action:

```php
// action.defaultadmin.php — view access is enough
if (!$this->CheckPermission(Holidays::VIEW_PERM)
    && !$this->CheckPermission(Holidays::MANAGE_PERM)) return;

// action.edit_holiday.php — requires manage access
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;

// action.delete_holiday.php — requires manage access
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;
```

### Cleaning Up Permissions

Remove all permissions in your `method.uninstall.php`:

```php
$this->RemovePermission(Holidays::MANAGE_PERM);
$this->RemovePermission(Holidays::VIEW_PERM);
```

### Frontend Actions

Frontend actions typically do not require permission checks since they are public-facing. However, if your module exposes frontend actions that modify data (e.g., a user profile editor), you should verify the user's identity through other means — such as checking the FrontEndUsers module session.

### Checklist

- Define permission names as class constants.
- Create permissions in `method.install.php`.
- Remove permissions in `method.uninstall.php`.
- Check `VisibleToAdminUser()` for navigation visibility.
- Check `CheckPermission()` at the top of every admin action file.

### Next Steps

Continue to [Data Validation](/modules/security/data-validation/) to learn how to verify that incoming data meets your expectations.
