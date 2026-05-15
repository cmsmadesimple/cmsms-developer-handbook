## Tutorial: Pretty URLs and Routes for Custom Modules

> **Note:** If search engine optimization is valuable to your site, you should ensure that your usage of the `detailpage` parameter is consistent throughout the website. When developing a module it may be better to use a preference instead of allowing the `detailpage` to be specified in the call to the module.

### SEO-Friendly URLs and Routes

Search Engine Friendly URLs (CMSMS calls them "Pretty URLs") are usually important to modules that display data on the website front-end — particularly if they are modules that will be released for use by the general public.

By default, module URLs look like `index.php?mact=Holidays,cntnt01,detail,0&hid=5`. Pretty URLs transform these into something like `/Holidays/99/5/Christmas`.

In a CMSMS module there are two parts to the "Pretty URL" problem:

1. **Generating** the pretty URL when rendering our HTML code.
2. **Routing** an incoming pretty URL to the proper module action.

---

### Step 1: Enable Pretty URLs on Your Website

If you have not done so already, follow the CMSMS configuration guide to enable pretty URLs on your website. Internal pretty URLs will be enough for this tutorial.

> **Note:** Remember to clear the CMSMS cache every time you adjust the `config.php` file.

---

### Step 2: Generate Pretty URLs

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

#### Discussion

This method handles the **generation** of pretty URLs. Here's what it does:

- Checks that we are creating a URL for the `detail` action and that we have the `hid` parameter.
- Loads the holiday object from the database.
- Builds a string containing the prefix `Holidays`, the page ID that we want to render the detail view on, the holiday ID, and a URL-safe string representing the name of the holiday.

The `munge_string_to_url()` function takes a holiday name and translates the characters until they are all safe URL characters.

This method is called automatically any time `Holidays::create_url()` is called for a frontend request. The `{cms_action_url}` Smarty plugin calls `Holidays::create_url()` for each URL we try to create. Therefore, every time we call `{cms_action_url}` to create a URL for the detail action, we will return a pretty URL.

---

### Step 3: Register the Route

Add route registration and a `junk` parameter to your `InitializeFrontend()` method:

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

#### Discussion

The modifications to `InitializeFrontend()` tell CMSMS how to handle incoming URL strings that match a certain pattern.

- We create a string parameter called `junk`. This is because the `get_pretty_url()` method appends the holiday name to the generated URL, and it must be handled. We already have the ability (via the `hid` parameter) to uniquely identify which holiday we want to display.
- `RegisterRoute()` tells CMSMS: for every incoming request where the URL matches this regular expression, call the `detail` action for the Holidays module, and parse out the `returnid` and `hid` parameters.

---

### Step 4: Update the Detail Template

Modify `templates/detail.tpl` to generate a canonical URL for SEO:

```smarty
{if $holiday}
  {cms_action_url action=detail hid=$holiday->id assign='canonical'}
  {$canonical=$canonical scope=global}
  <article class="holiday-detail">
    <!-- ... rest of template ... -->
  </article>
{/if}
```

The page template can then use `{$canonical}` in a `<link rel="canonical">` tag. Routes are just additional ways to access the same data — the old "un-pretty" URL still works to generate a detail view. That is why we add the canonical URL, so search engines know which URL is authoritative.

---

### Step 5: Bump the Version

Since we changed `InitializeFrontend()`, update the version in your module class:

```php
public function GetVersion() { return '0.1.1'; }
```

Then visit **Module Manager** and click "Upgrade" on the Holidays row.

---

### Testing

Visit your holidays summary page on the frontend. Hover over a detail link — you should see a pretty URL like:

```
https://yoursite.com/index.php/Holidays/99/5/Christmas
```

If you have mod\_rewrite configured, it will be:

```
https://yoursite.com/Holidays/99/5/Christmas
```

> **Note:** Your URL will look slightly different if you configured mod\_rewrite pretty URLs, and because your `returnid`, holiday ID, and holiday name may be different.

Clicking the link should display the detail view, and the old-style URL still works too.

---

### General Notes

- **Route uniqueness:** The regular expression defined in the route must be able to uniquely identify your module and all of the details to generate a view, without conflicting with another module. To help with this, module routes are usually prefixed with the module name. Without the prefix, a route of `99/2/Christmas` could refer to another module or to a content page. The system finds the first route that matches, which may not necessarily be yours.

- **Page ID requirement:** There must be some mechanism on every incoming route to determine a page ID. This can either be encoded in the URL itself or stored in some type of preference, but it must exist. Your decision on how to store the `returnid` depends on how your module will be used.

- **Simplifying the URL:** Although it is possible to remove the `hid` from our routes so that URLs look like `Holidays/<returnid>/<name>`, we would need to modify our code to ensure that the holiday name was unique for all holidays so that we could uniquely identify the holiday to display.

- **Default URL action:** If no action is specified as a parameter to the route, but a route is matched, then the magic module action `defaulturl` will be called.

- **Performance consideration:** In our `get_pretty_url()` method we load the `HolidayItem` specified by the `hid`. It is entirely probable that in a detail view that holiday item has already been loaded. This would result in the same data being loaded twice from the database. For a production-ready module, the developer should take steps to ensure that this does not happen, and utilize caching to optimize performance.

---

### Status Check

The module now generates and handles SEO-friendly URLs for detail views. Next, we'll add Search module integration, lazy loading, and AJAX previews. Continue to {cms\_selflink dir='next' text='Search, Lazy Loading, and AJAX'}.
