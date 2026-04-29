## Tutorial: Frontend Views

Now we add the frontend — a summary view that lists published holidays, and a detail view that shows a single holiday. This involves registering parameters, creating frontend actions, and building templates.

### Step 1: Register Frontend Parameters

Add the `InitializeFrontend()` and `InitializeAdmin()` methods to your `Holidays.module.php` class:

```php
public function InitializeFrontend()
{
    $this->RegisterModulePlugin();
    $this->SetParameterType('hid', CLEAN_INT);
    $this->SetParameterType('pagelimit', CLEAN_INT);
    $this->SetParameterType('detailpage', CLEAN_STRING);
}

public function InitializeAdmin()
{
    $this->CreateParameter('hid', null, $this->Lang('param_hid'));
    $this->CreateParameter('pagelimit', 1000, $this->Lang('param_pagelimit'));
    $this->CreateParameter('detailpage', null, $this->Lang('param_detailpage'));
}
```

`RegisterModulePlugin()` lets content editors use the shorthand `{Holidays}` tag instead of `{cms_module module=Holidays}`.

`SetParameterType()` registers each parameter with its expected type. Unregistered parameters are silently stripped from frontend requests.

### Step 2: The Default Frontend Action

Create `action.default.php` — the summary view:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

$limit = isset($params['pagelimit']) ? (int) $params['pagelimit'] : 1000;
$limit = max(1, $limit);

$detailpage = $returnid;
if (isset($params['detailpage'])) {
    $hm = CmsApp::get_instance()->GetHierarchyManager();
    $node = $hm->sureGetNodeByAlias($params['detailpage']);
    if (is_object($node)) $detailpage = $node->get_tag('id');
}

$query = new HolidayQuery(['published' => 1, 'limit' => $limit]);
$holidays = $query->GetMatches();

$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource('default.tpl'), null, null, $smarty
);
$tpl->assign('holidays', $holidays);
$tpl->assign('detailpage', $detailpage);
$tpl->display();
```

#### What this does:

- Reads the `pagelimit` parameter (defaults to 1000).
- Resolves the `detailpage` parameter — converts a page alias to a numeric page ID using the Hierarchy Manager. If not specified, detail links point to the current page.
- Queries only published holidays.
- Passes the holidays and detail page ID to the template.

### Step 3: The Summary Template

Create `templates/default.tpl`:

```smarty
&lt;div class="holiday-list"&gt;
  {foreach $holidays as $holiday}
    &lt;div class="holiday-item"&gt;
      &lt;a href="{cms_action_url action=detail hid=$holiday->id returnid=$detailpage}"&gt;
        {$holiday->name|escape}
      &lt;/a&gt;
      &amp;mdash;
      &lt;span class="date"&gt;{$holiday->the_date|date_format:'%x'}&lt;/span&gt;
    &lt;/div&gt;
  {foreachelse}
    &lt;p&gt;{$mod->Lang('sorry_noholidays')}&lt;/p&gt;
  {/foreach}
&lt;/div&gt;
```

### Step 4: The Detail Action

Create `action.detail.php`:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;
if (!isset($params['hid'])) return;

$holiday = HolidayItem::load_by_id((int) $params['hid']);

$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource('detail.tpl'), null, null, $smarty
);
$tpl->assign('holiday', $holiday);
$tpl->display();
```

### Step 5: The Detail Template

Create `templates/detail.tpl`:

```html
{if $holiday}
  &lt;article class="holiday-detail"&gt;
    &lt;h2&gt;{$holiday->name|escape}&lt;/h2&gt;
    &lt;p class="date"&gt;{$holiday->the_date|date_format:'%x'}&lt;/p&gt;
    &lt;div class="description"&gt;
      {$holiday->description}
    &lt;/div&gt;
  &lt;/article&gt;
{else}
  &lt;p&gt;{$mod->Lang('error_notfound')}&lt;/p&gt;
{/if}
```

### Step 6: Add Lang Strings

Add to `lang/en_US.php`:

```
$lang['sorry_noholidays'] = 'Sorry, we could not find any holidays that match the specified criteria';
$lang['error_notfound'] = 'The Holiday specified could not be displayed';
$lang['param_hid'] = 'Applicable only to the detail action, this parameter accepts the integer id of a holiday to display';
$lang['param_pagelimit'] = 'Applicable only to the default action, this parameter limits the number of holidays displayed';
$lang['param_detailpage'] = 'Applicable only to the default action, this parameter allows specifying an alternate page alias on which to display the detail results';
```

### Step 7: Call the Module from a Page

1. Create a new content page in CMSMS.
2. On the Options tab, check "Disable WYSIWYG editor on this page".
3. In the content area, enter:

```smarty
{Holidays pagelimit=10}
```

Visit the page on the frontend. You should see a list of your published holidays with links to detail views.

### Status Check

The module now has both admin and frontend functionality:

- Admin: add, edit, delete, list holidays.
- Frontend: summary view with links to detail views.
- The `detailpage` parameter lets you display details on a different page.
- The `pagelimit` parameter controls how many holidays are shown.

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
    ├── default.tpl
    ├── defaultadmin.tpl
    ├── detail.tpl
    └── edit_holiday.tpl
```

Next, we'll add SEO-friendly URLs. Continue to {cms\_selflink dir='next' text='Pretty URLs and Routes'}.
