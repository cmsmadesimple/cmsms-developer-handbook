## Frontend AJAX

AJAX on the frontend lets your module load content dynamically without a full page reload. Common use cases include live previews, infinite scroll, filtering results, and submitting forms without navigation. The approach builds on the same action and template system you already use for regular frontend views.

### How It Works

A frontend AJAX request is just a normal module action call with one key addition: the `showtemplate=false` parameter. This tells CMSMS to render only the module's output, stripping the surrounding page template (header, footer, navigation). The result is a clean HTML fragment or JSON response suitable for inserting into the page via JavaScript.

### Returning HTML Fragments

The simplest approach is to return a rendered Smarty template. You don't need a special "AJAX action" for this. Your existing detail or list action can serve double duty by accepting an alternate template parameter:

```php
<?php
// action.detail.php
if (!defined('CMS_VERSION')) exit;
if (!isset($params['hid'])) return;

$detailtemplate = isset($params['detailtemplate'])
    ? trim($params['detailtemplate'])
    : 'detail.tpl';

$item = MyItem::load_by_id((int) $params['hid']);

$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource($detailtemplate), null, null, $smarty
);
$tpl->assign('item', $item);
$tpl->display();
```

Create a lightweight template for the AJAX response:

```smarty
{* templates/ajax_detail.tpl *}
{if $item}
<div class="item-preview">
  <strong>{$item->name|cms_escape}</strong>
  <p>{$item->description|strip_tags|truncate:120}</p>
</div>
{/if}
```

Register the parameter in your module class:

```php
public function InitializeFrontend()
{
    $this->RegisterModulePlugin();
    $this->SetParameterType('hid', CLEAN_INT);
    $this->SetParameterType('detailtemplate', CLEAN_STRING);
}
```

### Returning JSON

For structured data responses, clear the output buffers, set the content type, and exit:

```php
<?php
// action.ajax_search.php
if (!defined('CMS_VERSION')) exit;

$query = isset($params['q']) ? trim($params['q']) : '';
$results = [];

if (strlen($query) >= 2) {
    $db = \cms_utils::get_db();
    $sql = 'SELECT id, name FROM '.CMS_DB_PREFIX.'module_mymodule WHERE name LIKE ? AND published=1 LIMIT 10';
    $rows = $db->GetArray($sql, ['%'.$query.'%']);
    if ($rows) $results = $rows;
}

$handlers = ob_list_handlers();
for ($i = 0; $i < count($handlers); $i++) { ob_end_clean(); }

header('Content-Type: application/json');
echo json_encode(['success' => true, 'results' => $results]);
exit;
```

If your module depends on CMSMSExt, replace the buffer clearing, header, and exit with:

```php
\xt_utils::send_ajax_and_exit(['success' => true, 'results' => $results]);
```

Register the action parameter:

```php
$this->SetParameterType('q', CLEAN_STRING);
```

### Generating AJAX URLs in Templates

Use `{cms_action_url}` with `forjs=1` to generate URLs suitable for JavaScript. The `forjs=1` parameter ensures the URL uses `&` instead of `&amp;` so it works correctly in JavaScript strings:

```smarty
{cms_action_url action=detail hid=$item->id returnid=$returnid
                forjs=1 detailtemplate='ajax_detail.tpl' assign='preview_url'}

{cms_action_url action=ajax_search returnid=$returnid forjs=1 assign='search_url'}
```

### The showtemplate=false Parameter

When you append `&showtemplate=false` to a frontend module URL, CMSMS renders only the content block output. The page template (everything outside `{content}`) is suppressed. This is what makes the response usable as an AJAX fragment.

You append it in JavaScript, not in the Smarty tag:

```javascript
var url = '{$preview_url}' + '&showtemplate=false';
```

### JavaScript: Loading HTML Fragments

Use jQuery's `.load()` method to fetch and insert HTML:

```smarty
{cms_action_url action=detail hid=$item->id returnid=$returnid
                forjs=1 detailtemplate='ajax_detail.tpl' assign='preview_url'}

<a class="preview-link" href="{$preview_url}" data-id="{$item->id}">Preview</a>
<div id="preview-area" style="display:none;"></div>

<script type="text/javascript">
$(document).ready(function(){
    $('a.preview-link').click(function(e){
        e.preventDefault();
        var url = $(this).attr('href') + '&showtemplate=false';
        $('#preview-area').load(url).show();
    });
});
</script>
```

### JavaScript: Working with JSON

Use `$.ajax()` for JSON endpoints:

```smarty
{cms_action_url action=ajax_search returnid=$returnid forjs=1 assign='search_url'}

<input type="text" id="search-input" placeholder="Search...">
<div id="search-results"></div>

<script type="text/javascript">
$(document).ready(function(){
    var timer;
    $('#search-input').on('keyup', function(){
        clearTimeout(timer);
        var q = $(this).val();
        if (q.length < 2) return;
        timer = setTimeout(function(){
            $.ajax({
                url: '{$search_url}' + '&showtemplate=false&{$actionid}q=' + encodeURIComponent(q),
                dataType: 'json',
                success: function(data) {
                    if (data.success) {
                        var html = '';
                        $.each(data.results, function(i, item){
                            html += '<div>' + item.name + '</div>';
                        });
                        $('#search-results').html(html);
                    }
                }
            });
        }, 300);
    });
});
</script>
```

### Prefixing Parameters with the Action ID

When sending parameters to a module action via AJAX, prefix them with `{$actionid}`. This is the same mechanism CMSMS uses for form submissions. It scopes the parameter to your module instance on the page:

```javascript
url: '{$ajax_url}' + '&showtemplate=false&{$actionid}item_id=' + id
```

Without the prefix, the parameter won't appear in the `$params` array in your action file.

### Submitting Forms via AJAX

For POST requests (saving data, submitting forms), use `$.ajax()` with `type: 'POST'`:

```smarty
{cms_action_url action=ajax_save returnid=$returnid forjs=1 assign='save_url'}

<form id="ajax-form">
    <input type="text" name="item_name" value="">
    <button type="button" id="save-btn">Save</button>
</form>
<div id="form-message"></div>

<script type="text/javascript">
$(document).ready(function(){
    $('#save-btn').click(function(){
        var data = {};
        data['{$actionid}item_name'] = $('input[name="item_name"]').val();
        data['showtemplate'] = 'false';

        $.ajax({
            url: '{$save_url}',
            type: 'POST',
            data: data,
            dataType: 'json',
            success: function(response) {
                if (response.success) {
                    $('#form-message').text('Saved!');
                } else {
                    $('#form-message').text(response.message);
                }
            }
        });
    });
});
</script>
```

> **Important:** Use `type="button"` on AJAX trigger buttons, not `type="submit"`. This prevents the form from submitting normally.

### Pretty URLs and AJAX

If your module uses pretty URLs via `get_pretty_url()`, you should skip pretty URL generation for AJAX requests. Pretty URLs trigger a redirect, which breaks AJAX calls:

```php
public function get_pretty_url($id, $action, $returnid = '', $params = array(), $inline = false)
{
    if ($action != 'detail' || !isset($params['hid'])) return;
    if (isset($params['detailtemplate'])) return; // skip for AJAX templates
    // ... normal pretty URL logic
}
```

### Security Considerations

- **Validate all input.** Frontend AJAX actions are publicly accessible. Never trust `$params` values without validation.
- **Use parameterized queries.** SQL injection applies equally to AJAX endpoints.
- **Rate limiting.** Consider whether your AJAX endpoint could be abused (e.g., a search endpoint hammered with requests).
- **Escape output.** Use `|cms_escape` in templates and `htmlspecialchars()` in JSON string values that will be rendered as HTML.
- **POST for mutations.** Use POST requests for any action that modifies data. GET requests should only read data.

### Ensuring jQuery is Available

CMSMS does not load jQuery on the frontend by default. Add `{cms_jquery}` to the `<head>` section of your page template:

```smarty
<head>
    {cms_jquery}
    ...
</head>
```

This loads the version of jQuery bundled with CMSMS.

### Complete Example

Here is a full working example of a module that displays a list of items with AJAX-powered preview:

**action.default.php** (summary view):

```php
<?php
if (!defined('CMS_VERSION')) exit;

$items = MyItem::load_published();

$tpl = $smarty->CreateTemplate($this->GetTemplateResource('default.tpl'), null, null, $smarty);
$tpl->assign('items', $items);
$tpl->assign('returnid', $returnid);
$tpl->display();
```

**action.detail.php** (serves both full detail and AJAX preview):

```php
<?php
if (!defined('CMS_VERSION')) exit;
if (!isset($params['hid'])) return;

$detailtemplate = isset($params['detailtemplate'])
    ? trim($params['detailtemplate'])
    : 'detail.tpl';

$item = MyItem::load_by_id((int) $params['hid']);

$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource($detailtemplate), null, null, $smarty
);
$tpl->assign('item', $item);
$tpl->display();
```

**templates/default.tpl**:

```smarty
<div class="item-list">
  {foreach $items as $item}
    {cms_action_url action=detail hid=$item->id returnid=$returnid
                    forjs=1 detailtemplate='ajax_detail.tpl' assign='preview_url'}
    <div class="item-row">
      <a href="{cms_action_url action=detail hid=$item->id returnid=$returnid}">
        {$item->name|cms_escape}
      </a>
      <a class="preview-link" href="{$preview_url}">Preview</a>
    </div>
  {/foreach}
</div>

<div id="preview-area" style="display:none; padding:10px; background:#f5f5f5; margin-top:15px;"></div>

<script type="text/javascript">
$(document).ready(function(){
    $('a.preview-link').click(function(e){
        e.preventDefault();
        var url = $(this).attr('href') + '&showtemplate=false';
        $('#preview-area').load(url).show(0).delay(5000).hide(0);
    });
});
</script>
```

**templates/ajax_detail.tpl**:

```smarty
{if $item}
<div class="item-preview">
  <strong>{$item->name|cms_escape}</strong>
  <p>{$item->description|strip_tags|truncate:200}</p>
</div>
{/if}
```

### Key Differences from Admin AJAX

| | Admin AJAX | Frontend AJAX |
|---|---|---|
| Permission check | Always required (`CheckPermission()`) | Not applicable (public) |
| URL generation | `{cms_action_url action=... forjs=1}` | Same |
| Suppress output | `SuppressAdminOutput()` or `ob_end_clean()` | `showtemplate=false` parameter |
| jQuery availability | Always loaded in admin | Requires `{cms_jquery}` in page template |
| Pretty URLs | Not applicable | Must be skipped for AJAX templates |
