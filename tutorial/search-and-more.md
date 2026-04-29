## Tutorial: Search, Lazy Loading, and AJAX

In this final chapter, we add three enhancements: integration with the CMSMS Search module, lazy loading for the admin, and an AJAX-powered preview on the frontend.

### Search Module Integration

The Search module can index your module's content so holidays appear in site search results. This is optional — if the Search module isn't installed, the code simply does nothing.

#### Index words when saving

Add this code to `action.edit_holiday.php`, after the `save()` call:

```php
$holiday->save();

// Update search index
$search = \cms_utils::get_search_module();
if (is_object($search)) {
    if (!$holiday->published) {
        $search->DeleteWords($this->GetName(), $holiday->id, 'holiday');
    } else {
        $search->AddWords($this->GetName(), $holiday->id, 'holiday',
            $holiday->name . ' ' . strip_tags($holiday->description));
    }
}

$this->SetMessage($this->Lang('holiday_saved'));
```

#### Remove words when deleting

Add this to `action.delete_holiday.php`, after the `delete()` call:

```php
$holiday->delete();

$search = \cms_utils::get_search_module();
if (is_object($search)) {
    $search->DeleteWords($this->GetName(), (int) $params['hid'], 'holiday');
}
```

#### Reindex all content

Add the `SearchReindex()` method to your `Holidays.module.php` class. The Search module calls this when rebuilding its index:

```php
public function SearchReindex(&$search_module)
{
    $query = new HolidayQuery(['published' => 1]);
    $matches = $query->GetMatches();
    if ($matches) {
        foreach ($matches as $holiday) {
            $search_module->AddWords($this->GetName(), $holiday->id, 'holiday',
                $holiday->name . ' ' . strip_tags($holiday->description));
        }
    }
}
```

#### Return search results

Add the `SearchResultWithParams()` method to display your module's items in search results:

```php
public function SearchResultWithParams($returnid, $articleid, $attr = '', $params = '')
{
    if ($attr != 'holiday') return;
    $holiday = HolidayItem::load_by_id((int) $articleid);
    if (!$holiday) return;

    $detailpage = $returnid;
    if (isset($params['detailpage'])) {
        $hm = CmsApp::get_instance()->GetHierarchyManager();
        $node = $hm->sureGetNodeByAlias($params['detailpage']);
        if (is_object($node)) $detailpage = $node->get_tag('id');
    }

    return [
        $this->GetFriendlyName(),
        $holiday->name,
        $this->create_url('cntnt01', 'detail', $detailpage, ['hid' => (int) $articleid]),
    ];
}
```

This returns an array of [module name, display text, URL] that the Search module uses to render the result.

### Lazy Loading

Add lazy loading for admin requests so the module isn't loaded into memory on every admin page:

```
public function LazyLoadAdmin() { return true; }
```

Bump the version and upgrade:

```
public function GetVersion() { return '0.1.2'; }
```
> **Note:** Our module cannot be lazy-loaded on the frontend because we call `RegisterModulePlugin()` and `RegisterRoute()` in `InitializeFrontend()`. These require the module to be loaded on every frontend request.

### AJAX Preview

Let's add a "Preview" link to the summary view that loads a brief holiday preview without a full page reload.

#### Create the AJAX detail template

Create `templates/ajax_detail.tpl`:

```html
{if $holiday}
&lt;div&gt;
  &lt;strong&gt;{$holiday->name|escape}&lt;/strong&gt; &amp;mdash; {$holiday->the_date|date_format:'%x'}
  &lt;p&gt;{$holiday->description|strip_tags|summarize}&lt;/p&gt;
&lt;/div&gt;
{/if}
```

#### Update the detail action to accept an alternate template

Modify `action.detail.php`:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;
if (!isset($params['hid'])) return;

$detailtemplate = isset($params['detailtemplate'])
    ? trim($params['detailtemplate'])
    : 'detail.tpl';

$holiday = HolidayItem::load_by_id((int) $params['hid']);

$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource($detailtemplate), null, null, $smarty
);
$tpl->assign('holiday', $holiday);
$tpl->display();
```

#### Register the new parameter

Add to `InitializeFrontend()`:

```php
$this->SetParameterType('detailtemplate', CLEAN_STRING);
```

#### Update the pretty URL method

Don't generate pretty URLs when a custom template is requested. Update `get_pretty_url()`:

```php
public function get_pretty_url($id, $action, $returnid = '', $params = array(), $inline = false)
{
    if ($action != 'detail' || !isset($params['hid'])) return;
    if (isset($params['detailtemplate'])) return; // no pretty URL for AJAX
    $holiday = HolidayItem::load_by_id((int) $params['hid']);
    if (!is_object($holiday)) return;
    return "Holidays/$returnid/{$params['hid']}/" . munge_string_to_url($holiday->name);
}
```

#### Update the summary template

Replace `templates/default.tpl` with:

```smarty
&lt;script type="text/javascript"&gt;
$(document).ready(function(){
    $('a.preview').click(function(e){
        e.preventDefault();
        var url = $(this).attr('href') + '&showtemplate=false';
        $('#preview_area').load(url).show(0).delay(5000).hide(0);
    });
});
&lt;/script&gt;

&lt;div id="preview_area" style="display: none; padding: 10px; background: #f5f5f5; margin-bottom: 15px;"&gt;&lt;/div&gt;

&lt;div class="holiday-list"&gt;
  {foreach $holidays as $holiday}
    &lt;div class="holiday-item"&gt;
      &lt;a href="{cms_action_url action=detail hid=$holiday->id returnid=$detailpage}"&gt;
        {$holiday->name|escape}
      &lt;/a&gt;
      &amp;mdash;
      &lt;a class="preview"
         href="{cms_action_url action=detail hid=$holiday->id returnid=$detailpage forjs=1 detailtemplate='ajax_detail.tpl'}"&gt;
        {$mod->Lang('preview')}
      &lt;/a&gt;
      &amp;mdash;
      &lt;span class="date"&gt;{$holiday->the_date|date_format:'%x'}&lt;/span&gt;
    &lt;/div&gt;
  {foreachelse}
    &lt;p&gt;{$mod->Lang('sorry_noholidays')}&lt;/p&gt;
  {/foreach}
&lt;/div&gt;
```

The "Preview" link loads the `ajax_detail.tpl` template via AJAX. The `showtemplate=false` parameter tells CMSMS to return only the module output without the page template wrapper. The preview appears for 5 seconds then hides.

#### Add the lang string

```
$lang['preview'] = 'Preview';
```
> **Note:** Make sure jQuery is available on your frontend. Add `{cms_jquery}` to the `<head>` of your page template if it's not already included.

### Final Status

Congratulations! Your Holidays module is complete. Here's everything it can do:

- Installs and uninstalls cleanly (database table, permission).
- Admin CRUD — add, edit, delete, and list holidays.
- Permission-controlled admin access.
- Frontend summary and detail views.
- Configurable page limit and detail page parameters.
- SEO-friendly pretty URLs with route registration.
- Search module integration (index, reindex, search results).
- Lazy loading on admin requests.
- AJAX-powered preview on the frontend.
- All text uses language strings for translation support.
- All SQL uses parameterized queries.
- MVC pattern — model classes in `lib/`, actions as controllers, Smarty templates as views.

### Final File Structure

```
modules/Holidays/
├── Holidays.module.php
├── action.default.php
├── action.defaultadmin.php
├── action.delete_holiday.php
├── action.detail.php
├── action.edit_holiday.php
├── method.install.php
├── method.uninstall.php
├── lang/
│   └── en_US.php
├── lib/
│   ├── class.HolidayItem.php
│   └── class.HolidayQuery.php
└── templates/
    ├── ajax_detail.tpl
    ├── default.tpl
    ├── defaultadmin.tpl
    ├── detail.tpl
    └── edit_holiday.tpl
```

### Taking It Further

This tutorial covered the fundamentals. Here are some ideas for extending the module:

- **Error handling** — Add try/catch blocks and validation error display.
- **Pagination** — Add page navigation to the admin list and frontend summary.
- **Categories** — Add a categories table and filtering.
- **Design Manager integration** — Register template types so admins can customize templates from the admin panel.
- **method.upgrade.php** — Add an upgrade routine for future schema changes.

For reference on any of these topics, see the corresponding chapters in the [Module Handbook](/modules/).
