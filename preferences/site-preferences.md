## Site and User Preferences

Beyond module-scoped preferences, CMSMS provides two additional preference systems: site preferences (global settings shared across the entire installation) and user preferences (settings tied to individual admin users).

### Site Preferences (cms\_siteprefs)

Site preferences are global key-value pairs not scoped to any module. They are stored in the site preferences table and accessible from anywhere in the system. Use them for settings that need to be shared across modules or accessed outside your module's context.

#### Reading and writing

```
// Set a site preference
cms_siteprefs::set('mymodule_global_setting', 'value');

// Get a site preference with a default
$val = cms_siteprefs::get('mymodule_global_setting', 'default_value');

// Check if a site preference exists
if (cms_siteprefs::exists('mymodule_global_setting')) {
    // ...
}

// Remove a site preference
cms_siteprefs::remove('mymodule_global_setting');

// Remove with LIKE matching (removes all keys starting with the prefix)
cms_siteprefs::remove('mymodule_', true);

// List all site preferences matching a prefix
$prefs = cms_siteprefs::list_by_prefix('mymodule_');
// Returns array of preference names, or null
```
> **Note:** Since site preferences are not namespaced to your module, always prefix your keys with your module name to avoid collisions (e.g., `holidays_detailpage` not `detailpage`).

#### When to use site preferences vs module preferences

| Use case | Recommendation |
| --- | --- |
| Module-specific settings (items per page, templates, toggles) | Module preferences (`$this->SetPreference()`) |
| Settings that other modules need to read | Site preferences (`cms_siteprefs`) |
| Settings accessed outside module scope (UDTs, plugins) | Site preferences (`cms_siteprefs`) |

#### cms\_siteprefs API reference

| Method | Description |
| --- | --- |
| `cms_siteprefs::get($key, $default)` | Get a site preference |
| `cms_siteprefs::set($key, $value)` | Set a site preference |
| `cms_siteprefs::exists($key)` | Test if a site preference exists |
| `cms_siteprefs::remove($key, $like)` | Remove a preference. Pass `true` for `$like` to match by prefix. |
| `cms_siteprefs::list_by_prefix($prefix)` | List preference names matching a prefix |

### User Preferences (cms\_userprefs)

User preferences are key-value pairs tied to individual admin users. Use them for per-user settings like display preferences, last-used filters, or UI state.

#### Current user

```
// Set a preference for the currently logged-in admin user
cms_userprefs::set('holidays_sort_order', 'date_desc');

// Get a preference for the current user
$sort = cms_userprefs::get('holidays_sort_order', 'date_asc');

// Check if it exists
if (cms_userprefs::exists('holidays_sort_order')) {
    // ...
}

// Remove it
cms_userprefs::remove('holidays_sort_order');

// Remove with LIKE matching
cms_userprefs::remove('holidays_', true);
```

#### Specific user

```
// Set a preference for a specific user by user ID
cms_userprefs::set_for_user($user_id, 'holidays_sort_order', 'date_desc');

// Get a preference for a specific user
$sort = cms_userprefs::get_for_user($user_id, 'holidays_sort_order', 'date_asc');

// Check if it exists for a specific user
if (cms_userprefs::exists_for_user($user_id, 'holidays_sort_order')) {
    // ...
}

// Get all preferences for a user
$all = cms_userprefs::get_all_for_user($user_id);
// Returns associative array: ['key1' => 'value1', 'key2' => 'value2']

// Remove a preference for a specific user
cms_userprefs::remove_for_user($user_id, 'holidays_sort_order');

// Remove ALL preferences for a specific user (omit key)
cms_userprefs::remove_for_user($user_id);
```

#### Practical example: remembering a filter

```
// action.defaultadmin.php

// Save the user's filter choice
if (isset($params['filter_category'])) {
    cms_userprefs::set('holidays_filter', $params['filter_category']);
}

// Restore the user's last filter
$filter = cms_userprefs::get('holidays_filter', '');

$tpl->assign('current_filter', $filter);
```

#### cms\_userprefs API reference

| Method | Description |
| --- | --- |
| `cms_userprefs::get($key, $default)` | Get a preference for the current user |
| `cms_userprefs::set($key, $value)` | Set a preference for the current user |
| `cms_userprefs::exists($key)` | Test if a preference exists for the current user |
| `cms_userprefs::remove($key, $like)` | Remove a preference for the current user |
| `cms_userprefs::get_for_user($uid, $key, $default)` | Get a preference for a specific user |
| `cms_userprefs::set_for_user($uid, $key, $value)` | Set a preference for a specific user |
| `cms_userprefs::exists_for_user($uid, $key)` | Test if a preference exists for a specific user |
| `cms_userprefs::remove_for_user($uid, $key, $like)` | Remove preferences for a specific user |
| `cms_userprefs::get_all_for_user($uid)` | Get all preferences for a specific user as an associative array |

### Cleaning Up

Module preferences are automatically cleaned up on uninstall if `AllowUninstallCleanup()` returns `true` (the default). However, site preferences and user preferences set by your module are not automatically removed — you must clean them up in `method.uninstall.php`:

```
// method.uninstall.php
cms_siteprefs::remove('holidays_', true);  // Remove all site prefs with this prefix

// User prefs are harder to clean up globally — consider if it's necessary
// They will be removed when the user account is deleted
```

### Next Steps

This completes the Preferences chapter. Continue to [Database Operations](/modules/database/) to learn how to create tables, query data, and manage schema migrations.
