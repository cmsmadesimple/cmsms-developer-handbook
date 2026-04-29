## Smarty Templates

Every module action that produces HTML output should do so through a Smarty template. Templates live in the `templates/` directory of your module and are loaded using the `GetTemplateResource()` method.

### Loading and Rendering a Template

The standard pattern in an action file:

```php
$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource('defaultadmin.tpl'), null, null, $smarty
);
$tpl->assign('items', $items);
$tpl->assign('total', $total);
$tpl->display();
```

#### What each part does:

- `GetTemplateResource('defaultadmin.tpl')` — converts the filename into a Smarty resource string that points to `templates/defaultadmin.tpl` in your module directory.
- `$smarty->CreateTemplate(..., null, null, $smarty)` — creates a new Smarty scope. The fourth argument passes the parent scope so that global variables remain accessible.
- `$tpl->assign('items', $items)` — passes data from your action to the template.
- `$tpl->display()` — processes and outputs the template.

> **Note:** Always create a new Smarty scope with `CreateTemplate()`. This prevents your variables from accidentally overwriting variables set by other module calls on the same page — especially important on the frontend where a module can be called multiple times.

### Template File Naming

By convention, template files match their action names:

| Action file | Template file |
| --- | --- |
| `action.defaultadmin.php` | `templates/defaultadmin.tpl` |
| `action.default.php` | `templates/default.tpl` |
| `action.edit_holiday.php` | `templates/edit_holiday.tpl` |
| `action.detail.php` | `templates/detail.tpl` |

This is a convention, not a requirement — an action can load any template file.

### Variables Automatically Available

CMSMS automatically assigns these variables to your template scope, even when creating a new scope:

| Variable | Description |
| --- | --- |
| `{$actionid}` | Unique action ID — prefix for all form field names |
| `{$returnid}` | Current page ID (empty for admin requests) |
| `{$actionmodule}` | Name of the module executing the action |
| `{$actionparams}` | The parameters array passed to the action |
| `{$mod}` | Reference to your module object — use for `{$mod->Lang('key')}` |

### Useful Smarty Syntax

#### Conditionals

```html
{if !empty($items)}
  &lt;!-- show items --&gt;
{else}
  &lt;p&gt;{$mod->Lang('no_items')}&lt;/p&gt;
{/if}

{if $item->published}
  &lt;span class="active"&gt;Published&lt;/span&gt;
{/if}
```

#### Loops

```smarty
{foreach $items as $item}
  &lt;tr&gt;
    &lt;td&gt;{$item->name|escape}&lt;/td&gt;
    &lt;td&gt;{$item->the_date|date_format:'%x'}&lt;/td&gt;
  &lt;/tr&gt;
{foreachelse}
  &lt;tr&gt;&lt;td colspan="2"&gt;{$mod->Lang('no_items')}&lt;/td&gt;&lt;/tr&gt;
{/foreach}
```

#### Modifiers

```
{* HTML escaping *}
{$value|escape}

{* Date formatting *}
{$timestamp|date_format:'%Y-%m-%d'}
{$timestamp|date_format:'%x'}  {* locale-aware *}

{* String truncation *}
{$description|truncate:100:'...'}

{* Strip HTML tags *}
{$html_content|strip_tags}

{* Summarize (CMSMS plugin) *}
{$description|strip_tags|summarize}
```

#### Variable assignment

```smarty
{* Assign a value to a Smarty variable *}
{assign var='page_title' value=$item->name}

{* Assign from a plugin *}
{cms_action_url action=edit_item hid=$item->id assign='edit_url'}
```

### CMSMS Smarty Plugins

Beyond standard Smarty syntax, CMSMS provides plugins specifically for module development:

| Plugin | Purpose |
| --- | --- |
| `{form_start}` / `{form_end}` | Create a module form with CSRF protection and hidden fields |
| `{cms_action_url}` | Generate a URL to a module action |
| `{admin_icon}` | Render an admin theme icon |
| `{cms_textarea}` | Create a WYSIWYG-enabled textarea |
| `{cms_yesno}` | Generate yes/no dropdown options |
| `{cms_module}` | Call a module from a template |
| `{cms_selflink}` | Create a link to a content page |
| `{cms_jquery}` | Include jQuery on the frontend |
| `{content}` | Render the default content block in a page template |
| `{content_module}` | Render a module-provided content block |

A full list of available Smarty plugins can be found in the CMSMS admin panel under Extensions > Tags.

### Next Steps

Continue to [Admin Templates](/modules/templates/admin-templates/) for conventions specific to admin panel templates.
