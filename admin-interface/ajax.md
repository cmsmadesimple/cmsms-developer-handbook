## AJAX in the Admin

Many modern admin interfaces load data asynchronously using AJAX. CMSMS supports this through regular module actions that suppress the admin theme wrapper and return JSON or HTML fragments.

### Creating an AJAX Action

An AJAX action is a normal action file that outputs data directly instead of rendering a full admin page. The key differences are: suppress the admin output, set the correct content type, and exit after output.

```php
&lt;?php
// action.ajax_check.php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(MyModule::MANAGE_PERM)) {
    echo json_encode(['success' => false, 'message' => 'Permission denied']);
    exit;
}

$item_id = isset($params['item_id']) ? (int) $params['item_id'] : 0;

$response = ['success' => false, 'message' => ''];

try {
    $item = MyItem::load_by_id($item_id);
    if (!$item) throw new \Exception('Item not found');

    $response['success'] = true;
    $response['data'] = [
        'id'   => $item->id,
        'name' => $item->name,
    ];
} catch (\Exception $e) {
    $response['message'] = $e->getMessage();
}

// Clear any output buffers
$handlers = ob_list_handlers();
for ($i = 0; $i < count($handlers); $i++) { ob_end_clean(); }

header('Content-Type: application/json');
echo json_encode($response);
exit;
```

#### Key points:

- Always check permissions — AJAX actions are still admin actions.
- Clear output buffers with `ob_end_clean()` to prevent the admin theme from wrapping your output.
- Set the `Content-Type` header explicitly.
- Call `exit` after output to prevent further processing.

#### Using CMSMSExt

If your module depends on [CMSMSExt](https://dev.cmsmadesimple.org/projects/cmsmsext), you can replace the buffer clearing, header, JSON encoding, and exit with a single call:

```
// Replaces ob_end_clean + header + json_encode + exit
\xt_utils::send_ajax_and_exit($response);
```

This handles output buffers, sets the correct content type, encodes the data, and exits cleanly.

### Using SuppressAdminOutput()

An alternative to manually clearing buffers is the `SuppressAdminOutput()` method, which tells CMSMS not to render the admin header, footer, or theme around your action's output:

```php
&lt;?php
// action.ajax_data.php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(MyModule::MANAGE_PERM)) return;

$this->SuppressAdminOutput();
header('Content-Type: application/json');
echo json_encode(['items' => $items]);
exit;

// Or with CMSMSExt:
$this->SuppressAdminOutput();
\xt_utils::send_ajax_and_exit(['items' => $items]);
```

### Generating AJAX URLs in Templates

Use `{cms_action_url}` to generate the URL for your AJAX endpoint, then call it from JavaScript:

```
{cms_action_url action=ajax_check assign='ajax_url'}

&lt;script type="text/javascript"&gt;
$(document).ready(function(){
    $('#check_btn').click(function(){
        var item_id = $('#item_select').val();
        $.ajax({
            url: '{$ajax_url}' + '&{$actionid}item_id=' + item_id + '&showtemplate=false',
            dataType: 'json',
            success: function(response) {
                if (response.success) {
                    // Handle success
                    $('#results').html(response.data.name);
                } else {
                    alert(response.message);
                }
            }
        });
    });
});
&lt;/script&gt;
```

#### The showtemplate=false parameter

`showtemplate=false` is a special CMSMS request parameter that tells the system to render only the content block output, without the surrounding page template. This is useful for both admin and frontend AJAX requests.

### Returning HTML Fragments

Instead of JSON, you can return rendered HTML from a Smarty template:

```php
&lt;?php
// action.ajax_detail.php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(MyModule::MANAGE_PERM)) return;

$item = MyItem::load_by_id((int) $params['item_id']);

$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource('ajax_detail.tpl'), null, null, $smarty
);
$tpl->assign('item', $item);
$tpl->display();
```

When called with `showtemplate=false`, only the template output is returned — no admin chrome.

### Frontend AJAX

The same pattern works for frontend AJAX. The PDF tutorial demonstrates a preview feature using jQuery:

```html
{cms_action_url action=detail hid=$holiday->id returnid=$detailpage
                forjs=1 detailtemplate='ajax_detail.tpl' assign='preview_url'}

&lt;a class="preview" href="{$preview_url}"&gt;{$mod->Lang('preview')}&lt;/a&gt;

&lt;script type="text/javascript"&gt;
$(document).ready(function(){
    $('a.preview').click(function(e){
        e.preventDefault();
        var url = $(this).attr('href') + '&showtemplate=false';
        $('#preview_area').load(url).show(0).delay(5000).hide(0);
    });
});
&lt;/script&gt;
```

The `forjs=1` parameter in `{cms_action_url}` triggers URL processing suitable for JavaScript use.

### Security Considerations

- Always check permissions in AJAX actions — they are accessible via URL just like any other action.
- Validate and sanitize all input parameters.
- For actions that modify data, consider using POST requests instead of GET.
- Return proper HTTP status codes for errors when appropriate.

### Next Steps

This completes the Admin Interface chapter. Continue to [Smarty Tags](/smarty-tags) to learn how to create frontend tags that content editors can use in pages and templates.
