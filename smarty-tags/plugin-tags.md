## Plugin Tags

There are two ways to call a module from a Smarty template: the generic `{cms_module}` tag, and a registered shorthand tag using the module's own name. Your module can also register entirely custom Smarty plugins.

### The {cms\_module} Tag

Every module with `IsPluginModule()` returning `true` can be called from a page template using the generic tag:

```smarty
{cms_module module=Holidays}

{* With parameters *}
{cms_module module=Holidays action=detail hid=5}

{* With a page limit *}
{cms_module module=Holidays limit=10 detailpage=holiday-detail}
```

This calls the module's `default` action (or the specified action) and renders the output in place of the tag.

### Registering the Module Name as a Tag

For a shorter, cleaner syntax, register your module name as a Smarty function plugin using `RegisterModulePlugin()` in `InitializeFrontend()`:

```php
protected function InitializeFrontend()
{
    $this->RegisterModulePlugin();
    $this->SetParameterType('hid', CLEAN_INT);
    $this->SetParameterType('limit', CLEAN_INT);
    $this->SetParameterType('detailpage', CLEAN_STRING);
}
```

Now content editors can use the shorter form:

```smarty
{Holidays}

{Holidays limit=10 detailpage=holiday-detail}

{Holidays action=detail hid=5}
```
> **Note:** Calling `RegisterModulePlugin()` in `InitializeFrontend()` means the module cannot be lazy-loaded on the frontend, because the callback runs after the module is already loaded. This is a trade-off — shorter tag syntax vs. memory usage.

### How Module Tags Work

When a module tag is encountered in a template, CMSMS:

1. Loads the module (if not already loaded).
2. Determines the action — either the `action` parameter or `default`.
3. Generates a unique action ID (`$id`) for this call.
4. Cleans the parameters through `SetParameterType()` registrations.
5. Calls `DoAction()` which includes the appropriate `action.*.php` file.
6. The action renders its template and the output replaces the tag.

### Registering Custom Smarty Plugins

Beyond the module tag itself, you can register additional Smarty plugins — function plugins, block plugins, or modifier plugins — that are tied to your module:

```php
protected function InitializeFrontend()
{
    $this->RegisterModulePlugin();

    // Register a custom function plugin: {holidays_count}
    $this->RegisterSmartyPlugin(
        'holidays_count',           // tag name
        'function',                 // type: function, block, compiler, modifier
        array($this->GetName(), 'smarty_count_plugin'),  // static callback
        true                        // cachable
    );
}

// The callback must be a static method
public static function smarty_count_plugin($params, $template)
{
    $db = \cms_utils::get_db();
    $sql = 'SELECT COUNT(*) FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE published = 1';
    return $db->GetOne($sql);
}
```

Content editors can then use:

```html
&lt;p&gt;We have {holidays_count} holidays in our database.&lt;/p&gt;
```

#### Plugin types

| Type | Smarty syntax | Description |
| --- | --- | --- |
| `function` | `{tagname param=value}` | Outputs a value. Most common type. |
| `block` | `{tagname}...{/tagname}` | Wraps content and can modify it. |
| `modifier` | `{$var|modifiername}` | Transforms a variable value. |
| `compiler` | `{tagname}` | Generates PHP code at compile time. Rarely used by modules. |

> **Note:** Modules that register additional Smarty plugins (beyond `RegisterModulePlugin`) cannot be lazy-loaded on either the frontend or admin side.

### Removing Smarty Plugins

If needed, you can unregister a plugin:

```php
$this->RemoveSmartyPlugin('holidays_count');
```

### Next Steps

Continue to [Content Block Tags](/modules/smarty-tags/content-block-tags/) to learn how to create custom content block types that appear in the page editor.
