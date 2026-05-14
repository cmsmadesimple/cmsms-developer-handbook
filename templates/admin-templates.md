## Admin Templates

Admin templates render the interface within the CMSMS admin console. They follow specific conventions for layout, forms, and navigation that ensure consistency with the admin theme.

### A Typical Admin List Template

```smarty
<!-- Action links -->
<div class="pageoptions">
  <a href="{cms_action_url action=edit_holiday}">
    {admin_icon icon='newobject.gif'} {$mod->Lang('add_holiday')}
  </a>
</div>

{if !empty($holidays)}
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
  {foreach $holidays as $holiday}
    {cms_action_url action=edit_holiday hid=$holiday->id assign='edit_url'}
    <tr>
      <td><a href="{$edit_url}">{$holiday->name|escape}</a></td>
      <td>{$holiday->the_date|date_format:'%x'}</td>
      <td><a href="{$edit_url}" title="{$mod->Lang('edit')}">
            {admin_icon icon='edit.gif'}</a></td>
      <td><a class="del_item"
             href="{cms_action_url action=delete_holiday hid=$holiday->id}"
             title="{$mod->Lang('delete')}">
            {admin_icon icon='delete.gif'}</a></td>
    </tr>
  {/foreach}
  </tbody>
</table>
{else}
  <p>{$mod->Lang('no_holidays')}</p>
{/if}
```

### A Typical Admin Form Template

```smarty
<h3>{$mod->Lang('edit_holiday')}</h3>

{form_start hid=$holiday->id}

<div class="pageoverflow">
  <p class="pageinput">
    <input type="submit" name="{$actionid}submit" value="{$mod->Lang('submit')}" />
    <input type="submit" name="{$actionid}cancel" value="{$mod->Lang('cancel')}" />
  </p>
</div>

<div class="pageoverflow">
  <p class="pagetext">{$mod->Lang('name')}:</p>
  <p class="pageinput">
    <input type="text" name="{$actionid}name" value="{$holiday->name|escape}" />
  </p>
</div>

<div class="pageoverflow">
  <p class="pagetext">{$mod->Lang('description')}:</p>
  <p class="pageinput">
    {cms_textarea prefix=$actionid name=description
                  value=$holiday->description enablewysiwyg=true}
  </p>
</div>

{form_end}
```

### Admin Template Conventions

- Place submit/cancel buttons at the top of the form so they are visible without scrolling.
- Use `pageoverflow` / `pagetext` / `pageinput` classes for form rows.
- Use `pagetable` for list tables, `pageicon` for icon columns.
- Use `pageoptions` for action link areas.
- Always use `{$mod->Lang('key')}` for text — never hardcode strings.
- Always escape user data with `{$var|escape}`.
- Prefix all form field names with `{$actionid}`.

### JavaScript in Admin Templates

jQuery and jQuery UI are automatically available in the admin. Add JavaScript directly in your template:

```smarty
<script type="text/javascript">
$(document).ready(function(){
    // Delete confirmation
    $('a.del_item').click(function(){
        return confirm('{$mod->Lang('confirm_delete')}');
    });

    // Sortable table rows
    $('table.pagetable tbody').sortable({
        update: function(event, ui) {
            // Handle reorder...
        }
    });
});
</script>
```

### Displaying Messages and Errors

Messages set with `SetMessage()` and `SetError()` are displayed automatically by the admin theme after a redirect. For inline errors (e.g., validation), pass them to the template:

```smarty
{* In the action *}
{* $tpl->assign('errors', $errors); *}

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

### Next Steps

Continue to [Frontend Templates](/docs/templates/frontend-templates) for conventions specific to frontend output.
