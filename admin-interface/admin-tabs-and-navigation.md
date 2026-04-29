## Admin Tabs and Navigation

CMSMS provides built-in support for tabbed interfaces in admin panels, control over where your module appears in the admin navigation, and the ability to add custom menu items and inject admin styles.

### Tabbed Interfaces

For modules with multiple sections (e.g., items list, settings, help), a tabbed interface keeps things organized. CMSMS provides two ways to create tabs: the CMSModule tab methods and the `cms_admin_tabs` utility class.

#### Using CMSModule tab methods

These methods are available on your module instance and output the HTML for a tabbed layout:

```php
&lt;?php
// action.defaultadmin.php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;

// Set the active tab (optional — remembers the last tab after redirect)
if (isset($params['active_tab'])) {
    $this->SetCurrentTab($params['active_tab']);
}

$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource('defaultadmin.tpl'), null, null, $smarty
);
$tpl->display();
```

In your template:

```
{* Tab headers *}
{$mod->StartTabHeaders()}
  {$mod->SetTabHeader('items', $mod->Lang('tab_items'))}
  {$mod->SetTabHeader('settings', $mod->Lang('tab_settings'))}
{$mod->EndTabHeaders()}

{* Tab content *}
{$mod->StartTabContent()}

  {$mod->StartTab('items')}
    &lt;!-- Items list content here --&gt;
  {$mod->EndTab()}

  {$mod->StartTab('settings')}
    &lt;!-- Settings form here --&gt;
  {$mod->EndTab()}

{$mod->EndTabContent()}
```

#### Using cms\_admin\_tabs from PHP

The static `cms_admin_tabs` class provides the same functionality and can be called directly from your action PHP code:

```php
// In your action file
cms_admin_tabs::set_current_tab('settings');

// Or build tab output in PHP before passing to Smarty
$tabs = cms_admin_tabs::start_tab_headers();
$tabs .= cms_admin_tabs::set_tab_header('items', $this->Lang('tab_items'));
$tabs .= cms_admin_tabs::set_tab_header('settings', $this->Lang('tab_settings'));
$tabs .= cms_admin_tabs::end_tab_headers();
$tpl->assign('tab_headers', $tabs);
```

In templates, use the CMSModule wrapper methods shown above — they are the standard approach.

#### Returning to a specific tab after redirect

When processing a form on a specific tab, redirect back to that tab:

```php
// In your action file
$this->SetMessage($this->Lang('settings_saved'));
$this->RedirectToAdminTab('settings');
```

### Admin Navigation Sections

The `GetAdminSection()` method controls where your module appears in the admin navigation menu:

```
public function GetAdminSection() { return 'extensions'; }
```

Available sections:

| Section | Location in Admin |
| --- | --- |
| `'content'` | Content menu |
| `'layout'` | Layout / Design menu |
| `'usersgroups'` | Users & Groups menu |
| `'extensions'` | Extensions menu (most common for modules) |
| `'siteadmin'` | Site Admin menu |
| `'myprefs'` | My Preferences menu |

### Custom Admin Menu Items

By default, CMSMS creates a single menu item for your module using `GetFriendlyName()` and the `defaultadmin` action. To add multiple menu items or customize the default one, override `GetAdminMenuItems()`:

```php
public function GetAdminMenuItems()
{
    $items = [];

    // Default menu item (built from module methods)
    $items[] = CmsAdminMenuItem::from_module($this);

    // Additional menu item pointing to a specific action
    $item = new CmsAdminMenuItem();
    $item->module = $this->GetName();
    $item->section = 'content';
    $item->title = $this->Lang('manage_categories');
    $item->action = 'manage_categories';
    $items[] = $item;

    return $items;
}
```

#### CmsAdminMenuItem properties

| Property | Type | Description |
| --- | --- | --- |
| `module` | string | The module name that hosts the action |
| `section` | string | The admin section (same values as GetAdminSection) |
| `title` | string | The text displayed in the navigation |
| `action` | string | The action to call when clicked |
| `url` | string | Optional — a custom URL. If not set, it is built from module + action |
| `icon` | string | Optional — URL to an icon for the menu item |
| `priority` | int | Sort priority (minimum 2) |

### Admin Header HTML

To inject CSS or JavaScript into the admin `<head>` when your module is active, override `GetHeaderHTML()`:

```php
public function GetHeaderHTML()
{
    $url = $this->GetModuleURLPath();
    return "&lt;link rel=\"stylesheet\" href=\"{$url}/css/admin.css\" /&gt;\n"
         . "&lt;script src=\"{$url}/js/admin.js\"&gt;&lt;/script&gt;\n";
}
```

### Admin Styles

To inject CSS for all admin pages (not just when your module is active), override `AdminStyle()`. This is called for every installed module on every admin request:

```
public function AdminStyle()
{
    return '.holidays-badge { color: green; font-weight: bold; }';
}
```
> **Note:** Use `AdminStyle()` sparingly — it runs on every admin page load. For module-specific styles, prefer `GetHeaderHTML()` which only runs when your module's action is displayed.

### Admin Theme CSS Classes

The CMSMS admin theme provides standard CSS classes for consistent layout. Use these in your admin templates:

| Class | Purpose |
| --- | --- |
| `pageoverflow` | Wrapper div for a form row (label + input) |
| `pagetext` | Paragraph containing the field label |
| `pageinput` | Paragraph containing the input field |
| `pageoptions` | Wrapper for action links (e.g., "Add new item") |
| `pagetable` | Standard table styling for list views |
| `pageicon` | Table column containing an icon (edit, delete) |
| `pageerror` | Error message container |

### jQuery in the Admin

The CMSMS admin console automatically includes jQuery and jQuery UI. You can use jQuery in your admin templates without including it yourself:

```
&lt;script type="text/javascript"&gt;
$(document).ready(function(){
    $('a.del_item').click(function(){
        return confirm('{$mod->Lang('confirm_delete')}');
    });
});
&lt;/script&gt;
```

### Next Steps

This completes the Admin Interface chapter. Continue to [Smarty Tags](/modules/smarty-tags/) to learn how to create frontend tags that content editors can use in pages and templates.
