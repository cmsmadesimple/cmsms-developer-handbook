## Tutorial: Admin CRUD

Now we build the admin interface: a list of holidays with add, edit, and delete functionality. This involves creating action files, Smarty templates, a query class, and adding lang strings.

### Step 1: The Query Class

Create `lib/class.HolidayQuery.php` to handle listing holidays with pagination:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

class HolidayQuery extends CmsDbQueryBase
{
    public function __construct($args = '')
    {
        parent::__construct($args);
        if (isset($this->_args['limit'])) {
            $this->_limit = (int) $this->_args['limit'];
        }
    }

    public function execute()
    {
        if (!is_null($this->_rs)) return;

        $sql = 'SELECT SQL_CALC_FOUND_ROWS H.*
                FROM ' . CMS_DB_PREFIX . 'mod_holidays H';

        if (isset($this->_args['published'])) {
            $tmp = $this->_args['published'];
            if ($tmp === 0) {
                $sql .= ' WHERE published = 0';
            } else if ($tmp === 1) {
                $sql .= ' WHERE published = 1';
            }
        }

        $sql .= ' ORDER BY the_date DESC';

        $db = \cms_utils::get_db();
        $this->_rs = $db->SelectLimit($sql, $this->_limit, $this->_offset);
        if ($db->ErrorMsg()) {
            throw new \CmsSQLErrorException($db->sql . ' -- ' . $db->ErrorMsg());
        }
        $this->_totalmatchingrows = $db->GetOne('SELECT FOUND_ROWS()');
    }

    public function &GetObject()
    {
        $obj = new HolidayItem();
        $obj->fill_from_array($this->fields);
        return $obj;
    }
}
```

This class extends `CmsDbQueryBase`, which handles pagination and result iteration. The `GetMatches()` method (inherited) returns an array of `HolidayItem` objects.

### Step 2: The Default Admin Action

Create `action.defaultadmin.php` — the entry point for the admin panel:

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

### Step 3: The Admin List Template

Create `templates/defaultadmin.tpl`:

```smarty
&lt;script type="text/javascript"&gt;
$(document).ready(function(){
    $('a.del_holiday').click(function(){
        return confirm('{$mod->Lang('confirm_delete')}');
    });
});
&lt;/script&gt;

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
      &lt;td&gt;&lt;a href="{$edit_url}" title="{$mod->Lang('edit')}"&gt;{$holiday->name|escape}&lt;/a&gt;&lt;/td&gt;
      &lt;td&gt;{$holiday->the_date|date_format:'%x'}&lt;/td&gt;
      &lt;td&gt;&lt;a href="{$edit_url}" title="{$mod->Lang('edit')}"&gt;
            {admin_icon icon='edit.gif'}&lt;/a&gt;&lt;/td&gt;
      &lt;td&gt;&lt;a class="del_holiday"
             href="{cms_action_url action=delete_holiday hid=$holiday->id}"
             title="{$mod->Lang('delete')}"&gt;
            {admin_icon icon='delete.gif'}&lt;/a&gt;&lt;/td&gt;
    &lt;/tr&gt;
  {/foreach}
  &lt;/tbody&gt;
&lt;/table&gt;
{/if}
```

### Step 4: The Edit/Add Action

Create `action.edit_holiday.php` — handles both adding new holidays and editing existing ones:

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

### Step 5: The Edit Form Template

Create `templates/edit_holiday.tpl`:

```sql
&lt;h3&gt;{$mod->Lang('add_holiday')}&lt;/h3&gt;

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

### Step 6: The Delete Action

Create `action.delete_holiday.php`:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(Holidays::MANAGE_PERM)) return;

if (isset($params['hid']) && $params['hid'] > 0) {
    $holiday = HolidayItem::load_by_id((int) $params['hid']);
    if ($holiday) {
        $holiday->delete();
        $this->SetMessage($this->Lang('holiday_deleted'));
    }
}
$this->RedirectToAdminTab();
```

### Step 7: Add Lang Strings

Add these to `lang/en_US.php`:

```
$lang['add_holiday'] = 'Create a New Holiday';
$lang['edit'] = 'Edit this';
$lang['delete'] = 'Delete this';
$lang['submit'] = 'Submit';
$lang['cancel'] = 'Cancel';
$lang['name'] = 'Name';
$lang['date'] = 'Date';
$lang['published'] = 'Published';
$lang['description'] = 'Description';
$lang['holiday_saved'] = 'This holiday is now saved';
$lang['holiday_deleted'] = 'This holiday is now deleted';
$lang['confirm_delete'] = 'Are you sure that you want to delete this holiday record?';
```

### Status Check

Your module now has a complete admin CRUD interface. Navigate to Extensions > Holidays to:

- Add new holidays with a name, date, published flag, and WYSIWYG description.
- See all holidays in a list table.
- Click a holiday name or the edit icon to edit it.
- Click the delete icon (with confirmation) to remove it.

```
modules/Holidays/
├── Holidays.module.php
├── action.defaultadmin.php
├── action.edit_holiday.php
├── action.delete_holiday.php
├── method.install.php
├── method.uninstall.php
├── lang/
│   └── en_US.php
├── lib/
│   ├── class.HolidayItem.php
│   └── class.HolidayQuery.php
└── templates/
    ├── defaultadmin.tpl
    └── edit_holiday.tpl
```

Next, we'll create the frontend views so visitors can see holidays on the website. Continue to {cms\_selflink dir='next' text='Frontend Views'}.
