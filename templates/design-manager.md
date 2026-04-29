## Design Manager Integration

By default, module templates are `.tpl` files stored in the `templates/` directory. This works well for development, but for modules intended for public distribution, integrating with the CMSMS Design Manager allows site administrators to customize templates from the admin panel (Layout > Design Manager) without editing files on the server.

### How It Works

Design Manager integration involves two classes:

- **CmsLayoutTemplateType** — defines a category of templates your module uses (e.g., "Summary View", "Detail View"). Registered during installation.
- **CmsLayoutTemplate** — an individual template stored in the database. Created from a template type with default content. Admins can edit, duplicate, and assign templates to designs.

Instead of loading templates from files with `GetTemplateResource()`, you load them from the database using the `CmsLayoutTemplate` API.

### Step 1: Register Template Types on Install

In `method.install.php`, create a template type for each view your module provides, and create a default template for each type:

```php
// method.install.php
if (!defined('CMS_VERSION')) exit;

$uid = get_userid(false);

// --- Register a "Summary" template type ---
try {
    $summary_type = new CmsLayoutTemplateType();
    $summary_type->set_originator($this->GetName());
    $summary_type->set_name('summary');
    $summary_type->set_dflt_flag(true);
    $summary_type->set_lang_callback('Holidays::page_type_lang_callback');
    $summary_type->set_content_callback('Holidays::reset_summary_defaults');
    $summary_type->reset_content_to_factory();
    $summary_type->save();

    // Create the default template
    $tpl = new CmsLayoutTemplate();
    $tpl->set_name('Holidays - Summary View');
    $tpl->set_owner($uid);
    $tpl->set_type($summary_type);
    $tpl->set_content($summary_type->get_dflt_contents());
    $tpl->set_type_dflt(true);
    $tpl->save();
} catch (\Exception $e) {
    // Handle error
}

// --- Register a "Detail" template type ---
try {
    $detail_type = new CmsLayoutTemplateType();
    $detail_type->set_originator($this->GetName());
    $detail_type->set_name('detail');
    $detail_type->set_dflt_flag(true);
    $detail_type->set_lang_callback('Holidays::page_type_lang_callback');
    $detail_type->set_content_callback('Holidays::reset_detail_defaults');
    $detail_type->reset_content_to_factory();
    $detail_type->save();

    $tpl = new CmsLayoutTemplate();
    $tpl->set_name('Holidays - Detail View');
    $tpl->set_owner($uid);
    $tpl->set_type($detail_type);
    $tpl->set_content($detail_type->get_dflt_contents());
    $tpl->set_type_dflt(true);
    $tpl->save();
} catch (\Exception $e) {
    // Handle error
}
```

Key points:

- `get_userid(false)` returns the current admin user ID — used to set the template owner.
- `reset_content_to_factory()` calls your content callback to populate the default content before saving the type.
- The template is created with `new CmsLayoutTemplate()` and each property is set explicitly — name, owner, type, content, and default flag.
- `set_type_dflt(true)` marks this template as the default for its type. Only one template per type can be the default.

### Step 2: Add Callbacks to Your Module Class

The template type uses callbacks for language translation, help text, and resetting to factory defaults:

```php
// In Holidays.module.php

public static function page_type_lang_callback($str)
{
    $mod = \cms_utils::get_module('Holidays');
    if ($mod) return $mod->Lang('tpltype_' . $str);
    return $str;
}

public static function reset_summary_defaults($type)
{
    $mod = \cms_utils::get_module('Holidays');
    if ($mod) {
        $fn = $mod->GetModulePath() . '/templates/default.tpl';
        if (is_file($fn)) return @file_get_contents($fn);
    }
    return '';
}

public static function reset_detail_defaults($type)
{
    $mod = \cms_utils::get_module('Holidays');
    if ($mod) {
        $fn = $mod->GetModulePath() . '/templates/detail.tpl';
        if (is_file($fn)) return @file_get_contents($fn);
    }
    return '';
}
```

The content callback receives the template type object and should return the default Smarty template content as a string. This is called when an admin clicks "Reset to Defaults" in the Design Manager.

### Step 3: Load Templates from the Database in Actions

Instead of loading from files, load the default (or user-selected) template from the database:

```php
// action.default.php — using Design Manager templates
if (!defined('CMS_VERSION')) exit;

// Load the default summary template for this module
$type_id = $this->GetName() . '::summary';
$tpl_obj = CmsLayoutTemplate::load_dflt_by_type($type_id);

// Or let the user choose a template via parameter
if (isset($params['template'])) {
    try {
        $tpl_obj = CmsLayoutTemplate::load($params['template']);
    } catch (\Exception $e) {
        $tpl_obj = CmsLayoutTemplate::load_dflt_by_type($type_id);
    }
}

// Get the Smarty resource string for this database template
$tpl = $smarty->CreateTemplate(
    $this->GetDatabaseResource($tpl_obj->get_name()), null, null, $smarty
);
$tpl->assign('holidays', $holidays);
$tpl->display();
```
> **Note:** Use `GetDatabaseResource()` instead of `GetTemplateResource()` when loading templates stored in the database.

### Step 4: Clean Up on Uninstall

```php
// method.uninstall.php
try {
    $types = CmsLayoutTemplateType::load_all_by_originator($this->GetName());
    if ($types) {
        foreach ($types as $type) {
            $templates = $type->get_template_list();
            if ($templates) {
                foreach ($templates as $tpl) {
                    $tpl->delete();
                }
            }
            $type->delete();
        }
    }
} catch (\Exception $e) {
    // Handle error
}
```

### CmsLayoutTemplateType Key Methods

| Method | Description |
| --- | --- |
| `set_originator($name)` | Set the module name that owns this type |
| `set_name($name)` | Set the template type name (e.g., 'summary', 'detail') |
| `set_dflt_flag($bool)` | Allow this type to have a default template |
| `set_dflt_contents($str)` | Set the default Smarty content for new templates of this type |
| `set_description($str)` | Set a description for the template type |
| `set_lang_callback($callable)` | Callback to translate the originator and name strings |
| `set_content_callback($callable)` | Callback to reset template content to factory defaults |
| `set_help_callback($callable)` | Callback to display help text when editing templates of this type |
| `save()` | Save the template type to the database |
| `create_new_template($name)` | Create a new CmsLayoutTemplate of this type |
| `get_template_list()` | Get all templates of this type |
| `load($val)` | Static — load by ID or 'Originator::name' string |
| `load_all_by_originator($name)` | Static — load all types for a module |

### CmsLayoutTemplate Key Methods

| Method | Description |
| --- | --- |
| `set_content($str)` | Set the Smarty template content |
| `get_content()` | Get the template content |
| `set_type_dflt($bool)` | Mark as the default template for its type |
| `save()` | Save to the database |
| `delete()` | Delete from the database |
| `load($val)` | Static — load by ID or name |
| `load_dflt_by_type($type)` | Static — load the default template for a type |
| `load_all_by_type($type)` | Static — load all templates of a type |
| `process()` | Process the template through Smarty and return the output |

### When to Use Design Manager Integration

- **Use it** for modules intended for public distribution on the Forge — it gives site administrators the ability to customize templates without FTP access.
- **Skip it** for private/internal modules where file-based templates are sufficient and simpler to maintain.

### Next Steps

This completes the Templates chapter. Continue to [Users and Permissions](/modules/users/) to learn how to work with admin users, groups, and the permission system.
