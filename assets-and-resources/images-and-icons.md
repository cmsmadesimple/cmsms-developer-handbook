## Images and Icons

Modules use images for admin interface icons, frontend content, and Forge listing screenshots. Each has different conventions.

### Admin Theme Icons

The CMSMS admin theme provides a set of standard icons. Use the `{admin_icon}` Smarty plugin to render them in your admin templates:

```smarty
{admin_icon icon='newobject.gif'}   {* Add / create *}
{admin_icon icon='edit.gif'}        {* Edit *}
{admin_icon icon='delete.gif'}      {* Delete *}
{admin_icon icon='info.gif'}        {* Information *}
{admin_icon icon='true.gif'}        {* Active / enabled *}
{admin_icon icon='false.gif'}       {* Inactive / disabled *}
{admin_icon icon='export.gif'}      {* Export *}
{admin_icon icon='import.gif'}      {* Import *}
{admin_icon icon='copy.gif'}        {* Copy / duplicate *}
{admin_icon icon='view.gif'}        {* View / preview *}
```

Always use theme icons for standard actions (edit, delete, add, etc.) rather than custom images. This ensures your module looks consistent with the rest of the admin panel and adapts to different admin themes.

### Custom Module Icons

If your module needs icons beyond what the admin theme provides, place them in an `images/` directory:

```
modules/YourModule/
└── images/
    ├── icon.gif
    └── custom-action.png
```

Reference them in templates using the module URL path:

```html
&lt;img src="{$mod->GetModuleURLPath()}/images/icon.gif" alt="Custom icon" /&gt;
```

Or from PHP:

```php
$icon_url = $this->GetModuleURLPath() . '/images/icon.gif';
```

### Module Navigation Icon

To display a custom icon next to your module in the admin navigation, you can set it via `GetAdminMenuItems()`:

```php
public function GetAdminMenuItems()
{
    $item = CmsAdminMenuItem::from_module($this);
    $item->icon = $this->GetModuleURLPath() . '/images/icon.gif';
    return [$item];
}
```

### Frontend Images

For images used in frontend templates:

```html
&lt;img src="{$mod->GetModuleURLPath()}/images/placeholder.png"
     alt="{$mod->Lang('placeholder_alt')}" /&gt;
```
> **Note:** For distributable modules, avoid bundling frontend images that impose a visual style. If you include sample images, document that they are placeholders the site developer should replace.

### User-Uploaded Images

If your module allows users to upload images (e.g., holiday photos), store them in the CMSMS uploads directory — not in the module directory:

```
$config = \cms_config::get_instance();
$upload_dir = $config['uploads_path'] . DIRECTORY_SEPARATOR . 'holidays';
$upload_url = $config['uploads_url'] . '/holidays';

// Create the directory if it doesn't exist
if (!is_dir($upload_dir)) @mkdir($upload_dir, 0755, true);

// Save the uploaded file
$safe_name = uniqid('holiday_') . '.' . $ext;
move_uploaded_file($_FILES['image']['tmp_name'], $upload_dir . DIRECTORY_SEPARATOR . $safe_name);

// Display in a template
// $tpl->assign('image_url', $upload_url . '/' . $safe_name);
```

Files in the module directory may be overwritten during module upgrades. The uploads directory is persistent and writable.

### Forge Screenshots

When submitting your module to the Forge, include screenshots that show your module in action. Place them in an `assets/` directory:

```
modules/YourModule/
└── assets/
    ├── screenshot_1.jpg
    ├── screenshot_2.jpg
    └── screenshot_3.jpg
```

Screenshots should show:

- The admin panel interface (list view, edit form).
- The frontend output (summary view, detail view).
- Any notable features (settings panel, special functionality).

Use descriptive filenames and keep file sizes reasonable (under 200KB each). JPG format is preferred for screenshots.

### Summary

| Asset type | Location | How to reference |
| --- | --- | --- |
| Admin theme icons | Built into the admin theme | `{admin_icon icon='edit.gif'}` |
| Custom module icons/images | `images/` | `{$mod->GetModuleURLPath()}/images/file.png` |
| Module CSS | `css/` | `GetHeaderHTML()` or template link tag |
| Module JavaScript | `js/` | `GetHeaderHTML()` or template script tag |
| User uploads | CMSMS uploads directory | `$config['uploads_url']` |
| Forge screenshots | `assets/` | Displayed on the Forge listing page |

### Next Steps

This completes the Assets and Resources chapter. For a complete walkthrough of building a module from scratch, see the [Tutorial: Building a Holidays Module](/modules/tutorial/).
