## Tutorial: The Model Class

Before building the admin interface, we create a PHP class to represent a single holiday record. This keeps our data access code clean, reusable, and separate from our controllers.

### Step 1: Create the lib Directory

Create a `lib/` directory inside `Holidays/`. CMSMS automatically loads classes from this directory.

### Step 2: Create class.HolidayItem.php

Create `lib/class.HolidayItem.php`:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

class HolidayItem
{
    private $_data = [
        'id' => null,
        'name' => null,
        'description' => null,
        'published' => null,
        'the_date' => null,
    ];

    public function __get($key)
    {
        if (array_key_exists($key, $this->_data)) {
            return $this->_data[$key];
        }
    }

    public function __set($key, $val)
    {
        switch ($key) {
            case 'name':
            case 'description':
                $this->_data[$key] = trim($val);
                break;
            case 'published':
                $this->_data[$key] = (bool) $val;
                break;
            case 'the_date':
                $this->_data[$key] = (int) $val;
                break;
        }
    }

    public function is_valid()
    {
        if (!$this->name) return false;
        if (!$this->the_date) return false;
        return true;
    }

    public function save()
    {
        if (!$this->is_valid()) return false;
        return ($this->id > 0) ? $this->update() : $this->insert();
    }

    protected function insert()
    {
        $db = \cms_utils::get_db();
        $sql = 'INSERT INTO ' . CMS_DB_PREFIX . 'mod_holidays
                (name, description, published, the_date) VALUES (?, ?, ?, ?)';
        $dbr = $db->Execute($sql, [
            $this->name, $this->description, $this->published, $this->the_date
        ]);
        if (!$dbr) return false;
        $this->_data['id'] = $db->Insert_ID();
        return true;
    }

    protected function update()
    {
        $db = \cms_utils::get_db();
        $sql = 'UPDATE ' . CMS_DB_PREFIX . 'mod_holidays
                SET name = ?, description = ?, published = ?, the_date = ?
                WHERE id = ?';
        $dbr = $db->Execute($sql, [
            $this->name, $this->description, $this->published, $this->the_date, $this->id
        ]);
        return ($dbr) ? true : false;
    }

    public function delete()
    {
        if (!$this->id) return false;
        $db = \cms_utils::get_db();
        $sql = 'DELETE FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE id = ?';
        $dbr = $db->Execute($sql, [$this->id]);
        if (!$dbr) return false;
        $this->_data['id'] = null;
        return true;
    }

    public function fill_from_array($row)
    {
        foreach ($row as $key => $val) {
            if (array_key_exists($key, $this->_data)) {
                $this->_data[$key] = $val;
            }
        }
    }

    public static function &load_by_id($id)
    {
        $id = (int) $id;
        $db = \cms_utils::get_db();
        $sql = 'SELECT * FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE id = ?';
        $row = $db->GetRow($sql, [$id]);
        $obj = null;
        if (is_array($row)) {
            $obj = new self();
            $obj->fill_from_array($row);
        }
        return $obj;
    }
}
```

#### Key points:

- PHP magic methods `__get` and `__set` provide controlled access to the data — the setter trims strings, casts booleans and integers.
- `is_valid()` checks that required fields are present before saving.
- `save()` decides whether to insert or update based on whether an ID exists.
- All SQL uses parameterized queries (`?` placeholders) to prevent SQL injection.
- `load_by_id()` is a static factory method that loads a holiday from the database.
- `fill_from_array()` populates the object from a database row — used by both `load_by_id()` and the query class we'll create later.
- The class uses `\cms_utils::get_db()` instead of `$this->GetDb()` because it's not a module class — it's a standalone model.

### Status Check

```
modules/Holidays/
├── Holidays.module.php
├── method.install.php
├── method.uninstall.php
├── lang/
│   └── en_US.php
└── lib/
    └── class.HolidayItem.php
```

We now have a model class for holiday records. Next, we'll build the admin interface to create, list, edit, and delete holidays. Continue to {cms\_selflink dir='next' text='Admin CRUD'}.
