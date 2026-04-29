## Tutorial: Pretty URLs and Routes

By default, module URLs look like `index.php?mact=Holidays,cntnt01,detail,0&hid=5`. Pretty URLs transform these into something like `/Holidays/99/5/Christmas`. This involves two parts: generating the pretty URL, and routing incoming requests back to the correct action.

> **Note:** Pretty URLs must be enabled in your CMSMS installation. Check Site Admin > Global Settings > Pretty URLs. Internal pretty URLs are sufficient for this tutorial.

### Step 1: Generate Pretty URLs

Add the `get_pretty_url()` method to your `Holidays.module.php` class:

```php
public function get_pretty_url($id, $action, $returnid = '', $params = array(), $inline = false)
{
    if ($action != 'detail' || !isset($params['hid'])) return;
    $holiday = HolidayItem::load_by_id((int) $params['hid']);
    if (!is_object($holiday)) return;
    return "Holidays/$returnid/{$params['hid']}/" . munge_string_to_url($holiday->name);
}
```

This method is called automatically by `{cms_action_url}` and `create_url()` when generating links. It returns a URL segment like `Holidays/99/5/Christmas` — the module name, the page ID, the holiday ID, and a URL-safe version of the holiday name.

### Step 2: Register the Route

Add route registration and a "junk" parameter to your `InitializeFrontend()` method:

```php
public function InitializeFrontend()
{
    $this->RegisterModulePlugin();
    $this->SetParameterType('hid', CLEAN_INT);
    $this->SetParameterType('pagelimit', CLEAN_INT);
    $this->SetParameterType('detailpage', CLEAN_STRING);
    $this->SetParameterType('junk', CLEAN_STRING);

    $this->RegisterRoute(
        '/Holidays\/(?P<returnid>[0-9]+)\/(?P<hid>[0-9]+)\/(?P<junk>.*?)$/',
        array('action' => 'detail')
    );
}
```

#### What this does:

- The `junk` parameter captures the holiday name slug in the URL — we don't use it (we identify the holiday by `hid`), but it must be registered so CMSMS doesn't strip it.
- `RegisterRoute()` tells CMSMS: "When an incoming URL matches this regex, call the `detail` action and extract `returnid` and `hid` from the URL."
- The route is prefixed with `Holidays/` to avoid conflicts with other modules.

### Step 3: Bump the Version

Since we changed `InitializeFrontend()`, update the version in your module class:

```
public function GetVersion() { return '0.1.1'; }
```

Then visit Extensions > Module Manager and click "Upgrade" on the Holidays row.

### Testing

Visit your holidays summary page on the frontend. Hover over a detail link — you should see a pretty URL like:

```
https://yoursite.com/index.php/Holidays/99/5/Christmas
```

If you have mod\_rewrite configured, it will be:

```
https://yoursite.com/Holidays/99/5/Christmas
```

Clicking the link should display the detail view, and the old-style URL still works too.

### Canonical URL

For SEO, add a canonical URL to your `templates/detail.tpl`:

```
{if $holiday}
  {cms_action_url action=detail hid=$holiday->id assign='canonical'}
  {$canonical=$canonical scope=global}
  &lt;article class="holiday-detail"&gt;
    &lt;!-- ... rest of template ... --&gt;
  &lt;/article&gt;
{/if}
```

The page template can then use `{$canonical}` in a `<link rel="canonical">` tag.

### Status Check

The module now generates and handles SEO-friendly URLs for detail views. Next, we'll add Search module integration, lazy loading, and AJAX previews. Continue to {cms\_selflink dir='next' text='Search, Lazy Loading, and AJAX'}.
