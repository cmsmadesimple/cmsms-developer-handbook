## Roles and Permissions

CMSMS uses a simple but effective permission model: admin users belong to groups, and permissions are assigned to groups. A user has a permission if any of their groups has that permission.

### How It Works

- **Admin users** — individual accounts that can log into the CMSMS admin panel.
- **Groups** — collections of users. Every user belongs to at least one group.
- **Permissions** — named capabilities (e.g., `manage_holidays`). Assigned to groups, not directly to users.

When you call `CheckPermission('manage_holidays')`, CMSMS checks whether the currently logged-in user belongs to any group that has that permission.

> **Note:** Members of the "Admin" group (the default super-admin group) automatically have all permissions without needing them explicitly assigned.

### Working with Users

The `User` and `UserOperations` classes provide access to admin user data:

```
// Get the currently logged-in user ID
$uid = get_userid(false);

// Load a user object
$userops = UserOperations::get_instance();
$user = $userops->LoadUserByID($uid);

echo $user->username;
echo $user->email;
echo $user->firstname . ' ' . $user->lastname;
```

### Working with Groups

```
// Get all groups
$groupops = GroupOperations::get_instance();
$groups = $groupops->LoadGroups();

// Check if a user is in a specific group
$group = $groupops->LoadGroupByName('Editor');
if ($group) {
    $members = $groupops->GetGroupMembers($group->id);
    // $members is an array of user IDs
}
```

### Checking Permissions

From within a module, use the `CheckPermission()` method:

```php
// Check if the current user has a specific permission
if ($this->CheckPermission('manage_holidays')) {
    // User has access
}

// Common pattern at the top of admin actions
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;
```

From outside a module context, use the `CmsPermission` class or check group membership directly.

### Next Steps

Continue to [Custom Permissions](/modules/users/custom-permissions/) to learn how to create and manage your own module permissions.
