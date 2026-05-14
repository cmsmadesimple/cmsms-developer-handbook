## Images and Icons

Modules use images for admin interface icons and frontend content. This page covers runtime image assets — the images your module uses while running. For Forge listing assets (icons, banners, screenshots), see [Module Assets](../cmsms-forge/module-assets.md).

### Admin Theme Icons

The CMSMS admin theme provides a set of standard icons for common actions. Use the `{admin_icon}` Smarty plugin to render them in your admin templates:

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

### Custom Module Images

If your module needs images beyond what the admin theme provides (e.g., custom action icons or decorative elements), place them in an `images/` directory:

```
modules/YourModule/
└── images/
    ├── custom-action.png
    └── placeholder.png
```

Reference them in templates using the module URL path:

```smarty
<img src="{$mod->GetModuleURLPath()}/images/custom-action.png" alt="Custom action" />
```

Or from PHP:

```php
$icon_url = $this->GetModuleURLPath() . '/images/custom-action.png';
```

### Frontend Images

For images used in frontend templates:

```smarty
<img src="{$mod->GetModuleURLPath()}/images/placeholder.png"
     alt="{$mod->Lang('placeholder_alt')}" />
```

> **Note:** For distributable modules, avoid bundling frontend images that impose a visual style. If you include sample images, document that they are placeholders the site developer should replace.

### User-Uploaded Images

If your module allows users to upload images, store them in the CMSMS uploads directory — **not** in the module directory. Files in the module directory are overwritten during upgrades.

```php
$config = \cms_config::get_instance();
$upload_dir = $config['uploads_path'] . DIRECTORY_SEPARATOR . 'modulename';
$upload_url = $config['uploads_url'] . '/modulename';

// Create the directory if it doesn't exist
if (!is_dir($upload_dir)) @mkdir($upload_dir, 0755, true);

// Save the uploaded file
$safe_name = uniqid('item_') . '.' . $ext;
move_uploaded_file($_FILES['image']['tmp_name'], $upload_dir . DIRECTORY_SEPARATOR . $safe_name);

// Use in a template
// $tpl->assign('image_url', $upload_url . '/' . $safe_name);
```

### Summary

| Asset type | Location | How to reference |
| --- | --- | --- |
| Admin theme icons | Built into the admin theme | `{admin_icon icon='edit.gif'}` |
| Custom module images | `images/` | `{$mod->GetModuleURLPath()}/images/file.png` |
| User uploads | CMSMS uploads directory | `$config['uploads_url']` |
| Forge assets (icon, banner, screenshots) | `assets/` | See [Module Assets](../cmsms-forge/module-assets.md) |

### Next Steps

For loading CSS and JavaScript, see [Loading CSS and JavaScript](loading-css-and-javascript.md). For Forge listing assets with size requirements, see [Module Assets](../cmsms-forge/module-assets.md).
