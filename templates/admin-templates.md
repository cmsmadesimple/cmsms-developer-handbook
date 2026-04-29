## Admin Templates

Admin templates render the interface within the CMSMS admin console. They follow specific conventions for layout, forms, and navigation that ensure consistency with the admin theme.

### A Typical Admin List Template

```smarty
&lt;!-- Action links --&gt;
&lt;div class="pageoptions"&gt;
  &lt;a href="{cms_action_url action=edit_holiday}"&gt;
    {admin_icon icon='newobject.gif'} {$mod->Lang('add_holiday')}
  &lt;/a&gt;
&lt;/div&gt;

{if !empty($holidays)}
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
  {foreach $holidays as $holiday}
    {cms_action_url action=edit_holiday hid=$holiday->id assign='edit_url'}
    &lt;tr&gt;
      &lt;td&gt;&lt;a href="{$edit_url}"&gt;{$holiday->name|escape}&lt;/a&gt;&lt;/td&gt;
      &lt;td&gt;{$holiday->the_date|date_format:'%x'}&lt;/td&gt;
      &lt;td&gt;&lt;a href="{$edit_url}" title="{$mod->Lang('edit')}"&gt;
            {admin_icon icon='edit.gif'}&lt;/a&gt;&lt;/td&gt;
      &lt;td&gt;&lt;a class="del_item"
             href="{cms_action_url action=delete_holiday hid=$holiday->id}"
             title="{$mod->Lang('delete')}"&gt;
            {admin_icon icon='delete.gif'}&lt;/a&gt;&lt;/td&gt;
    &lt;/tr&gt;
  {/foreach}
  &lt;/tbody&gt;
&lt;/table&gt;
{else}
  &lt;p&gt;{$mod->Lang('no_holidays')}&lt;/p&gt;
{/if}
```

### A Typical Admin Form Template

```smarty
&lt;h3&gt;{$mod->Lang('edit_holiday')}&lt;/h3&gt;

{form_start hid=$holiday->id}

&lt;div class="pageoverflow"&gt;
  &lt;p class="pageinput"&gt;
    &lt;input type="submit" name="{$actionid}submit" value="{$mod->Lang('submit')}" /&gt;
    &lt;input type="submit" name="{$actionid}cancel" value="{$mod->Lang('cancel')}" /&gt;
  &lt;/p&gt;
&lt;/div&gt;

&lt;div class="pageoverflow"&gt;
  &lt;p class="pagetext"&gt;{$mod->Lang('name')}:&lt;/p&gt;
  &lt;p class="pageinput"&gt;
    &lt;input type="text" name="{$actionid}name" value="{$holiday->name|escape}" /&gt;
  &lt;/p&gt;
&lt;/div&gt;

&lt;div class="pageoverflow"&gt;
  &lt;p class="pagetext"&gt;{$mod->Lang('description')}:&lt;/p&gt;
  &lt;p class="pageinput"&gt;
    {cms_textarea prefix=$actionid name=description
                  value=$holiday->description enablewysiwyg=true}
  &lt;/p&gt;
&lt;/div&gt;

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

```
&lt;script type="text/javascript"&gt;
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
&lt;/script&gt;
```

### Displaying Messages and Errors

Messages set with `SetMessage()` and `SetError()` are displayed automatically by the admin theme after a redirect. For inline errors (e.g., validation), pass them to the template:

```smarty
{* In the action *}
{* $tpl->assign('errors', $errors); *}

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

### Next Steps

Continue to [Frontend Templates](/modules/templates/frontend-templates/) for conventions specific to frontend output.
