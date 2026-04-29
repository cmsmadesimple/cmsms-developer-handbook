## Loading CSS and JavaScript

Modules can include their own stylesheets and JavaScript files for both admin and frontend use. The approach differs between the two contexts.

### Directory Structure

Place static assets in dedicated directories within your module:

```
modules/YourModule/
├── css/
│   ├── admin.css
│   └── frontend.css
├── js/
│   ├── admin.js
│   └── frontend.js
└── ...
```

### Admin: GetHeaderHTML()

To include CSS or JavaScript on admin pages when your module's action is displayed, override `GetHeaderHTML()` in your module class:

```php
public function GetHeaderHTML()
{
    $url = $this->GetModuleURLPath();
    $out = '&lt;link rel="stylesheet" href="' . $url . '/css/admin.css" /&gt;' . "\n";
    $out .= '&lt;script src="' . $url . '/js/admin.js"&gt;&lt;/script&gt;' . "\n";
    return $out;
}
```

This method is called only when one of your module's admin actions is being displayed — not on every admin page.

### Admin: AdminStyle()

To inject CSS on every admin page (not just your module's actions), override `AdminStyle()`. This is called for every installed module on every admin request:

```
public function AdminStyle()
{
    return '.holidays-status { font-weight: bold; color: green; }';
}
```
> **Note:** Use `AdminStyle()` sparingly — it runs on every admin page load. For module-specific styles, prefer `GetHeaderHTML()`.

### Admin: Inline in Templates

You can also include assets directly in your admin Smarty templates:

```
&lt;link rel="stylesheet" href="{$mod->GetModuleURLPath()}/css/admin.css" /&gt;
&lt;script src="{$mod->GetModuleURLPath()}/js/admin.js"&gt;&lt;/script&gt;
```

This approach is simpler but less clean than `GetHeaderHTML()` — the assets end up in the page body rather than the `<head>`.

### Frontend: In Templates

For frontend templates, include assets using the module URL path:

```
&lt;link rel="stylesheet" href="{$mod->GetModuleURLPath()}/css/frontend.css" /&gt;
&lt;script src="{$mod->GetModuleURLPath()}/js/frontend.js"&gt;&lt;/script&gt;
```

However, for distributable modules, think carefully before including frontend CSS — see the best practices section below.

### jQuery

#### Admin

jQuery and jQuery UI are automatically available in the CMSMS admin console. You do not need to include them — just use `$` in your admin templates:

```
&lt;script type="text/javascript"&gt;
$(document).ready(function(){
    // jQuery is already loaded
});
&lt;/script&gt;
```

#### Frontend

jQuery is not automatically included on the frontend. If your module needs it, instruct the site developer to add `{cms_jquery}` to the `<head>` of their page template:

```
&lt;head&gt;
  {cms_jquery}
  ...
&lt;/head&gt;
```

Do not include jQuery yourself from a CDN or your module directory — this can cause conflicts with other modules or the site's existing jQuery installation.

### Conditional Loading

If your module has heavy JavaScript or CSS that is only needed for certain actions, load it conditionally:

```php
// In your action file
$this->GetHeaderHTML(); // loads default assets

// Or add action-specific assets
$url = $this->GetModuleURLPath();
$headerhtml = '&lt;script src="' . $url . '/js/chart-library.js"&gt;&lt;/script&gt;';
// Pass to template
$tpl->assign('extra_head', $headerhtml);
```

### Best Practices for Distributable Modules

- **Admin CSS is fine.** Include styles for your admin panel — they only load when your module is active.
- **Avoid frontend CSS in distributable modules.** You don't know what CSS framework or design the site uses. Provide minimal, semantic HTML and let the site developer style it. If you must include sample CSS, make it optional.
- **Don't bundle jQuery.** Use the CMSMS-provided jQuery in admin, and document that `{cms_jquery}` is needed on the frontend.
- **Don't bundle large libraries.** If your module needs a charting library, date picker, or other large dependency, document it as a requirement and load it conditionally.
- **Minify production assets.** Ship minified CSS and JS for performance, but keep the source files available for developers.

### Next Steps

Continue to {cms\_selflink dir='next' text='Images and Icons'} to learn about working with admin theme icons, custom images, and Forge screenshots.
