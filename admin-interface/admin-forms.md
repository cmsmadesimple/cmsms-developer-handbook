## Admin Forms and Smarty Plugins

CMSMS provides a set of Smarty plugins specifically designed for building admin forms. These plugins handle action IDs, CSRF tokens, URL generation, and admin theme integration automatically.

### Form Plugins

#### {form\_start} and {form\_end}

The `{form_start}` plugin creates a `<form>` tag with all required hidden fields — module name, action ID, return ID, and CSRF token. The `{form_end}` plugin closes it.

```smarty
{* Basic form — submits back to the current action *}
{form_start}
  <!-- form fields -->
{form_end}

{* Form that submits to a specific action *}
{form_start action='save_settings'}
  <!-- form fields -->
{form_end}

{* Form with additional hidden parameters *}
{form_start action='edit_holiday' hid=$holiday->id}
  <!-- form fields -->
{form_end}

{* Form with file upload support *}
{form_start action='upload_file' enctype='multipart/form-data'}
  <!-- file input -->
{form_end}
```

Parameters passed to `{form_start}` (like `hid`) are automatically prefixed with `{$actionid}` and included as hidden fields.

#### {cms\_action\_url}

Generates a URL to a module action. Use it for links, not forms:

```html
{* Link to an action *}
<a href="{cms_action_url action=edit_holiday hid=$holiday->id}">Edit</a>

{* Assign to a variable for reuse *}
{cms_action_url action=edit_holiday hid=$holiday->id assign='edit_url'}
<a href="{$edit_url}">{$holiday->name}</a>

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
<select name="{$actionid}published">
  {cms_yesno selected=$holiday->published}
</select>
```

#### Standard HTML inputs

For text inputs, selects, checkboxes, and other standard fields, use plain HTML with the `{$actionid}` prefix:

```sql
<!-- Text input -->
<input type="text" name="{$actionid}name" value="{$holiday->name|escape}" />

<!-- Date input -->
<input type="date" name="{$actionid}the_date"
       value="{$holiday->the_date|date_format:'%Y-%m-%d'}" />

<!-- Select dropdown -->
<select name="{$actionid}category">
  {foreach $categories as $key => $label}
    <option value="{$key}" {if $key == $item->category}selected{/if}>
      {$label|escape}
    </option>
  {/foreach}
</select>

<!-- Submit and cancel buttons -->
<input type="submit" name="{$actionid}submit" value="{$mod->Lang('submit')}" />
<input type="submit" name="{$actionid}cancel" value="{$mod->Lang('cancel')}" />
```
> **Note:** Always prefix every input name with `{$actionid}`. This is how CMSMS routes form data back to the correct module action. Without the prefix, your parameters will not appear in the `$params` array.

### Admin Style Guide

The CMSMS admin theme provides standard CSS classes for consistent form layout. Using these classes ensures your module looks consistent with the rest of the admin panel.

#### Form row layout

```html
<div class="pageoverflow">
  <p class="pagetext">{$mod->Lang('name')}:</p>
  <p class="pageinput">
    <input type="text" name="{$actionid}name" value="{$item->name|escape}" />
  </p>
</div>
```

#### Action links area

```html
<div class="pageoptions">
  <a href="{cms_action_url action=edit_item}">
    {admin_icon icon='newobject.gif'} {$mod->Lang('add_item')}
  </a>
</div>
```

#### List table

```smarty
<table class="pagetable">
  <thead>
    <tr>
      <th>{$mod->Lang('name')}</th>
      <th>{$mod->Lang('date')}</th>
      <th class="pageicon"></th>
      <th class="pageicon"></th>
    </tr>
  </thead>
  <tbody>
  {foreach $items as $item}
    {cms_action_url action=edit_item hid=$item->id assign='edit_url'}
    <tr>
      <td><a href="{$edit_url}">{$item->name|escape}</a></td>
      <td>{$item->the_date|date_format:'%x'}</td>
      <td><a href="{$edit_url}" title="{$mod->Lang('edit')}">
            {admin_icon icon='edit.gif'}</a></td>
      <td><a class="del_item"
             href="{cms_action_url action=delete_item hid=$item->id}"
             title="{$mod->Lang('delete')}">
            {admin_icon icon='delete.gif'}</a></td>
    </tr>
  {/foreach}
  </tbody>
</table>
```

#### Error and message display

```smarty
{* Display validation errors *}
{if !empty($errors)}
<div class="pageerror">
  <ul>
  {foreach $errors as $error}
    <li>{$error}</li>
  {/foreach}
  </ul>
</div>
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

Continue to [Admin Tabs and Navigation](/docs/admin-interface/admin-tabs-and-navigation) to learn how to create tabbed interfaces and customize your module's position in the admin navigation.
