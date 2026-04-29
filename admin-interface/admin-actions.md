## Admin Actions

In CMSMS, controllers are called "actions." Each action is a separate PHP file in your module directory that handles a specific request. When an admin action is requested, CMSMS loads the corresponding file and executes it within the scope of your module class.

### The defaultadmin Action

The `action.defaultadmin.php` file is the entry point for your module's admin panel. It is called when a user clicks your module in the admin navigation, or when no specific action is requested:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;

$query = new HolidayQuery();
$holidays = $query->GetMatches();

$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource('defaultadmin.tpl'), null, null, $smarty
);
$tpl->assign('holidays', $holidays);
$tpl->display();
```

Every admin action should start with the two security checks: the `CMS_VERSION` guard and a permission check.

### Action File Naming

Action files follow the pattern `action.actionname.php` and are placed in the root of your module directory:

| File | Purpose |
| --- | --- |
| `action.defaultadmin.php` | Default admin action — entry point from the admin navigation |
| `action.default.php` | Default frontend action — entry point from a page template call |
| `action.edit_holiday.php` | Custom action — edit a holiday record |
| `action.delete_holiday.php` | Custom action — delete a holiday record |

### Variables Available in Actions

CMSMS passes several variables into every action file:

| Variable | Type | Description |
| --- | --- | --- |
| `$smarty` | object | A Smarty template object representing the current Smarty scope. Not necessarily the global Smarty instance. |
| `$action` | string | The name of the current action being executed. |
| `$id` | string | The unique module-action ID. For admin requests this is always `'m1_'`. On the frontend, it is generated to allow multiple calls to the same module on one page. |
| `$returnid` | int|empty | The numeric page ID being rendered. Always empty for admin requests (there is no "page" concept in the admin). |
| `$params` | array | Input parameters passed to the action — from form submissions, URL parameters, or the module call. For frontend requests, only registered parameters appear here. |
| `$db` | object | A reference to the global database connection object (convenience). |
| `$gCms` | object | A reference to the CmsApp application object. Provided for backward compatibility — prefer `CmsApp::get_instance()` or the `cmsms()` shorthand in new code. |

### Smarty Variables in Templates

CMSMS automatically provides these variables to your Smarty templates, even when creating a new scope:

| Variable | Description |
| --- | --- |
| `{$actionid}` | Same as `$id` in the action file. Used to prefix form field names. |
| `{$returnid}` | Same as `$returnid` in the action file. |
| `{$actionmodule}` | The name of the module currently executing the action. |
| `{$actionparams}` | Same as `$params` in the action file. |
| `{$mod}` | A reference to your module object. Use it to call `{$mod->Lang('key')}` and other module methods from templates. |

### Rendering Templates

The recommended pattern is to create a new Smarty scope, assign your data, and display:

```php
$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource('edit_holiday.tpl'), null, null, $smarty
);
$tpl->assign('holiday', $holiday);
$tpl->display();
```

Creating a new scope prevents your variables from accidentally overwriting variables set by other actions or the parent template.

### Form Processing

A typical admin action handles both displaying a form and processing its submission. Use the `$params` array to detect which button was clicked:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;

$holiday = new HolidayItem();

// Load existing record if editing
if (isset($params['hid']) && $params['hid'] > 0) {
    $holiday = HolidayItem::load_by_id((int) $params['hid']);
}

// Handle cancel
if (isset($params['cancel'])) {
    $this->RedirectToAdminTab();
}

// Handle form submission
if (isset($params['submit'])) {
    $holiday->name = trim($params['name']);
    $holiday->published = cms_to_bool($params['published']);
    $holiday->the_date = strtotime($params['the_date']);
    $holiday->description = $params['description'];
    $holiday->save();

    $this->SetMessage($this->Lang('holiday_saved'));
    $this->RedirectToAdminTab();
}

// Display the form
$tpl = $smarty->CreateTemplate(
    $this->GetTemplateResource('edit_holiday.tpl'), null, null, $smarty
);
$tpl->assign('holiday', $holiday);
$tpl->display();
```

### The Corresponding Template

```sql
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
  &lt;p class="pagetext"&gt;{$mod->Lang('date')}:&lt;/p&gt;
  &lt;p class="pageinput"&gt;
    &lt;input type="date" name="{$actionid}the_date"
           value="{$holiday->the_date|date_format:'%Y-%m-%d'}" /&gt;
  &lt;/p&gt;
&lt;/div&gt;

&lt;div class="pageoverflow"&gt;
  &lt;p class="pagetext"&gt;{$mod->Lang('published')}:&lt;/p&gt;
  &lt;p class="pageinput"&gt;
    &lt;select name="{$actionid}published"&gt;
      {cms_yesno selected=$holiday->published}
    &lt;/select&gt;
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

#### Key points about admin templates:

- `{form_start}` creates the `<form>` tag with all required hidden fields (module name, action ID, CSRF token). Pass additional parameters like `hid` to include them as hidden fields.
- Every input field name must be prefixed with `{$actionid}` so CMSMS can route the data back to your action.
- Use `{$mod->Lang('key')}` for all user-facing text.
- `{cms_yesno}` generates a yes/no dropdown. `{cms_textarea}` creates a WYSIWYG-enabled textarea.
- Use the CSS classes `pageoverflow`, `pagetext`, `pageinput`, and `pageoptions` — these are styles the CMSMS admin theme understands.

### Creating Links Between Actions

Use the `{cms_action_url}` Smarty plugin to generate URLs to other actions:

```html
&lt;!-- Link to the edit action --&gt;
&lt;a href="{cms_action_url action=edit_holiday hid=$holiday->id}"&gt;
  {admin_icon icon='edit.gif'} {$mod->Lang('edit')}
&lt;/a&gt;

&lt;!-- Link to add a new item --&gt;
&lt;div class="pageoptions"&gt;
  &lt;a href="{cms_action_url action=edit_holiday}"&gt;
    {admin_icon icon='newobject.gif'} {$mod->Lang('add_holiday')}
  &lt;/a&gt;
&lt;/div&gt;

&lt;!-- Assign URL to a variable for reuse --&gt;
{cms_action_url action=edit_holiday hid=$holiday->id assign='edit_url'}
&lt;a href="{$edit_url}"&gt;{$holiday->name}&lt;/a&gt;
```

The `{cms_action_url}` plugin automatically knows the module name and generates the correct URL. The `{admin_icon}` plugin renders admin theme icons.

### Redirecting

After processing a form, always redirect to prevent duplicate submissions:

```php
// Redirect back to the default admin action
$this->RedirectToAdminTab();

// Redirect to a specific tab
$this->RedirectToAdminTab('settings');

// Redirect to a different action
$this->Redirect($id, 'edit_holiday', '', ['hid' => $holiday->id]);
```

### Messages and Errors

Set messages before redirecting — they are displayed on the next page load:

```php
// Success message (green)
$this->SetMessage($this->Lang('holiday_saved'));

// Error message (red)
$this->SetError($this->Lang('error_saving'));
```

### Suppressing Admin Output

If your action needs to output raw data (e.g., JSON for AJAX, a file download), suppress the admin theme wrapper:

```php
// In your action file
$this->SuppressAdminOutput();
header('Content-Type: application/json');
echo json_encode($data);
exit;
```

### Next Steps

Continue to [Admin Tabs and Navigation](/modules/admin-interface/admin-tabs-and-navigation/) to learn how to create tabbed interfaces and customize your module's position in the admin navigation.
