## Frontend Templates

Frontend templates render your module's output on the public website. Unlike admin templates, frontend templates have no prescribed style guide — the website developer controls the HTML and CSS. This means your templates should be minimal, semantic, and easy to customize.

### Summary and Detail Views

Most content modules follow a two-view pattern:

- **Summary view** (`default.tpl`) — displays a list of items with links to individual detail pages.
- **Detail view** (`detail.tpl`) — displays a single item in full.

#### Summary template example

```smarty
<div class="holiday-list">
  {foreach $holidays as $holiday}
    <div class="holiday-item">
      <a href="{cms_action_url action=detail hid=$holiday->id returnid=$detailpage}">
        {$holiday->name|escape}
      </a>
      <span class="date">{$holiday->the_date|date_format:'%x'}</span>
    </div>
  {foreachelse}
    <p>{$mod->Lang('no_holidays')}</p>
  {/foreach}
</div>
```

#### Detail template example

```html
{if $holiday}
  <article class="holiday-detail">
    <h2>{$holiday->name|escape}</h2>
    <p class="date">{$holiday->the_date|date_format:'%x'}</p>
    <div class="description">
      {$holiday->description}
    </div>
  </article>
{else}
  <p>{$mod->Lang('error_notfound')}</p>
{/if}
```

### The Detail Page Parameter

Summary views often link to detail views on a different content page. The `detailpage` parameter controls which page renders the detail view:

```
{* In the page content *}
{Holidays limit=10 detailpage=holiday-detail}

{* In the action file *}
$detailpage = $returnid;
if (isset($params['detailpage'])) {
    $hm = CmsApp::get_instance()->GetHierarchyManager();
    $node = $hm->sureGetNodeByAlias($params['detailpage']);
    if (is_object($node)) $detailpage = $node->get_tag('id');
}
$tpl->assign('detailpage', $detailpage);
```

If no `detailpage` is specified, the detail view renders on the same page as the summary (`$returnid`).

### Design Considerations for Distributable Modules

If your module will be shared on the Forge, keep these guidelines in mind:

- **Keep templates minimal.** Provide basic, semantic HTML without imposing a CSS framework. Website developers will style it themselves.
- **Don't include CSS or images for frontend templates.** This forces your styling preferences on the developer. Provide sample templates that are functional but visually sparse.
- **Use semantic HTML.** Use `<article>`, `<section>`, `<time>`, and other semantic elements where appropriate.
- **Use CSS classes, not inline styles.** Let the developer target your elements with their own stylesheet.
- **Document the available Smarty variables.** List what variables your action assigns to the template so developers know what they can use when customizing.

### Alternate Templates

Allow users to specify an alternate template via a parameter:

```php
// In your action file
$template = isset($params['detailtemplate'])
    ? trim($params['detailtemplate'])
    : 'detail.tpl';

$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource($template), null, null, $smarty
);
```

This lets content editors use different templates for different contexts:

```smarty
{* Full detail view *}
{Holidays action=detail hid=5}

{* Compact detail for AJAX preview *}
{Holidays action=detail hid=5 detailtemplate='ajax_detail.tpl'}
```

### Design Manager Integration

For modules intended for public distribution, consider integrating with the CMSMS Design Manager. This allows site administrators to manage and customize your module's templates from within the admin panel (Layout > Design Manager) without editing files directly.

Design Manager integration involves registering your templates as template types and storing them in the database rather than as files. This is an advanced topic covered in the CMSMS API documentation for `CmsLayoutTemplate` and `CmsLayoutTemplateType`.

### The showtemplate Parameter

`showtemplate=false` is a special CMSMS request parameter that renders only the content block output without the surrounding page template. This is useful for AJAX requests:

```
{* Generate a URL for AJAX use *}
{cms_action_url action=detail hid=$holiday->id returnid=$detailpage
                forjs=1 detailtemplate='ajax_detail.tpl' assign='ajax_url'}

<script>
$('#preview').load('{$ajax_url}' + '&showtemplate=false');
</script>
```

### Canonical URLs

For SEO, set a canonical URL in your detail template so search engines know the preferred URL for the content:

```smarty
{if $holiday}
  {cms_action_url action=detail hid=$holiday->id assign='canonical'}
  {$canonical=$canonical scope=global}
  <!-- The page template can then use {$canonical} in a <link rel="canonical"> tag -->
{/if}
```

### Next Steps

This completes the Templates chapter. Continue to [Users and Permissions](/docs/users/users-overview) to learn how to work with admin users, groups, and the permission system.
