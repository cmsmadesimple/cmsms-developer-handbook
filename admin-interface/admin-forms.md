## Admin Forms and Smarty Plugins

CMSMS provides a set of Smarty plugins specifically designed for building admin forms. These plugins handle action IDs, CSRF tokens, URL generation, and admin theme integration automatically.

### Form Plugins

#### {form\_start} and {form\_end}

The `{form_start}` plugin creates a `<form>` tag with all required hidden fields — module name, action ID, return ID, and CSRF token. The `{form_end}` plugin closes it.

```smarty
{* Basic form — submits back to the current action *}
{form_start}
  &lt;!-- form fields --&gt;
{form_end}

{* Form that submits to a specific action *}
{form_start action='save_settings'}
  &lt;!-- form fields --&gt;
{form_end}

{* Form with additional hidden parameters *}
{form_start action='edit_holiday' hid=$holiday->id}
  &lt;!-- form fields --&gt;
{form_end}

{* Form with file upload support *}
{form_start action='upload_file' enctype='multipart/form-data'}
  &lt;!-- file input --&gt;
{form_end}
```

Parameters passed to `{form_start}` (like `hid`) are automatically prefixed with `{$actionid}` and included as hidden fields.

#### {cms\_action\_url}

Generates a URL to a module action. Use it for links, not forms:

```html
{* Link to an action *}
&lt;a href="{cms_action_url action=edit_holiday hid=$holiday->id}"&gt;Edit&lt;/a&gt;

{* Assign to a variable for reuse *}
{cms_action_url action=edit_holiday hid=$holiday->id assign='edit_url'}
&lt;a href="{$edit_url}"&gt;{$holiday->name}&lt;/a&gt;

{* Link to a frontend action on a specific page *}
{cms_action_url action=detail hid=$holiday->id returnid=$detailpage}
```

The plugin automatically determines the module name. You can pass any additional parameters and they will be included in the URL.

#### {admin\_icon}

Renders an icon from the admin theme:

```smarty
{admin_icon icon='newobject.gif'}
{admin_icon icon='edit.gif'}
{admin_icon icon='delete.gif'}
{admin_icon icon='info.gif'}
{admin_icon icon='export.gif'}
{admin_icon icon='import.gif'}
```

Common icons available in the admin theme:

| Icon | Typical use |
| --- | --- |
| `newobject.gif` | Add / create new item |
| `edit.gif` | Edit an item |
| `delete.gif` | Delete an item |
| `info.gif` | Information / help |
| `true.gif` | Active / enabled state |
| `false.gif` | Inactive / disabled state |
| `export.gif` | Export action |
| `import.gif` | Import action |
| `copy.gif` | Copy / duplicate |
| `view.gif` | View / preview |

### Input Plugins

#### {cms\_textarea}

Creates a textarea that automatically integrates with the installed WYSIWYG editor (e.g., MicroTiny):

```
{cms_textarea prefix=$actionid name='description'
             value=$holiday->description enablewysiwyg=true}

{* Without WYSIWYG — plain textarea *}
{cms_textarea prefix=$actionid name='notes' value=$item->notes}
```

The `prefix` parameter should always be `$actionid`. The `name` parameter becomes the field name (prefixed automatically).

#### {cms\_yesno}

Generates a yes/no dropdown:

```sql
&lt;select name="{$actionid}published"&gt;
  {cms_yesno selected=$holiday->published}
&lt;/select&gt;
```

#### Standard HTML inputs

For text inputs, selects, checkboxes, and other standard fields, use plain HTML with the `{$actionid}` prefix:

```sql
&lt;!-- Text input --&gt;
&lt;input type="text" name="{$actionid}name" value="{$holiday->name|escape}" /&gt;

&lt;!-- Date input --&gt;
&lt;input type="date" name="{$actionid}the_date"
       value="{$holiday->the_date|date_format:'%Y-%m-%d'}" /&gt;

&lt;!-- Select dropdown --&gt;
&lt;select name="{$actionid}category"&gt;
  {foreach $categories as $key => $label}
    &lt;option value="{$key}" {if $key == $item->category}selected{/if}&gt;
      {$label|escape}
    &lt;/option&gt;
  {/foreach}
&lt;/select&gt;

&lt;!-- Submit and cancel buttons --&gt;
&lt;input type="submit" name="{$actionid}submit" value="{$mod->Lang('submit')}" /&gt;
&lt;input type="submit" name="{$actionid}cancel" value="{$mod->Lang('cancel')}" /&gt;
```
> **Note:** Always prefix every input name with `{$actionid}`. This is how CMSMS routes form data back to the correct module action. Without the prefix, your parameters will not appear in the `$params` array.

### Admin Style Guide

The CMSMS admin theme provides standard CSS classes for consistent form layout. Using these classes ensures your module looks consistent with the rest of the admin panel.

#### Form row layout

```html
&lt;div class="pageoverflow"&gt;
  &lt;p class="pagetext"&gt;{$mod->Lang('name')}:&lt;/p&gt;
  &lt;p class="pageinput"&gt;
    &lt;input type="text" name="{$actionid}name" value="{$item->name|escape}" /&gt;
  &lt;/p&gt;
&lt;/div&gt;
```

#### Action links area

```html
&lt;div class="pageoptions"&gt;
  &lt;a href="{cms_action_url action=edit_item}"&gt;
    {admin_icon icon='newobject.gif'} {$mod->Lang('add_item')}
  &lt;/a&gt;
&lt;/div&gt;
```

#### List table

```smarty
&lt;table class="pagetable"&gt;
  &lt;thead&gt;
    &lt;tr&gt;
      &lt;th&gt;{$mod->Lang('name')}&lt;/th&gt;
      &lt;th&gt;{$mod->Lang('date')}&lt;/th&gt;
      &lt;th class="pageicon"&gt;&lt;/th&gt;
      &lt;th class="pageicon"&gt;&lt;/th&gt;
    &lt;/tr&gt;
  &lt;/thead&gt;
  &lt;tbody&gt;
  {foreach $items as $item}
    {cms_action_url action=edit_item hid=$item->id assign='edit_url'}
    &lt;tr&gt;
      &lt;td&gt;&lt;a href="{$edit_url}"&gt;{$item->name|escape}&lt;/a&gt;&lt;/td&gt;
      &lt;td&gt;{$item->the_date|date_format:'%x'}&lt;/td&gt;
      &lt;td&gt;&lt;a href="{$edit_url}" title="{$mod->Lang('edit')}"&gt;
            {admin_icon icon='edit.gif'}&lt;/a&gt;&lt;/td&gt;
      &lt;td&gt;&lt;a class="del_item"
             href="{cms_action_url action=delete_item hid=$item->id}"
             title="{$mod->Lang('delete')}"&gt;
            {admin_icon icon='delete.gif'}&lt;/a&gt;&lt;/td&gt;
    &lt;/tr&gt;
  {/foreach}
  &lt;/tbody&gt;
&lt;/table&gt;
```

#### Error and message display

```smarty
{* Display validation errors *}
{if !empty($errors)}
&lt;div class="pageerror"&gt;
  &lt;ul&gt;
  {foreach $errors as $error}
    &lt;li&gt;{$error}&lt;/li&gt;
  {/foreach}
  &lt;/ul&gt;
&lt;/div&gt;
{/if}
```

### CSS Class Reference

| Class | Element | Purpose |
| --- | --- | --- |
| `pageoverflow` | div | Wrapper for a form row (label + input pair) |
| `pagetext` | p | Field label within a form row |
| `pageinput` | p | Input field within a form row |
| `pageoptions` | div | Action links area (e.g., "Add new item") |
| `pagetable` | table | Standard list/data table |
| `pageicon` | th / td | Narrow column for an icon (edit, delete) |
| `pageerror` | div | Error message container |

The admin theme also includes a basic 12-column grid system for more complex form layouts.

### Next Steps

Continue to [Admin Tabs and Navigation](/modules/admin-interface/admin-tabs-and-navigation/) to learn how to create tabbed interfaces and customize your module's position in the admin navigation.
