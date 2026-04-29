## Schema Management

As your module evolves, you'll need to add columns, create new tables, and migrate data. CMSMS handles this through the `method.upgrade.php` file. This page also covers the `CmsDbQueryBase` class for building reusable, paginated query objects.

### Incremental Upgrades

The `method.upgrade.php` file receives the previously installed version in the `$oldversion` variable. Use `version_compare()` to apply changes incrementally:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

$db = $this->GetDb();
$dict = NewDataDictionary($db);

// Version 1.1: add a category column
if (version_compare($oldversion, '1.1', '&lt;')) {
    $sqlarray = $dict->AddColumnSQL(
        CMS_DB_PREFIX . 'mod_holidays',
        'category C(100)'
    );
    $dict->ExecuteSQLArray($sqlarray);
}

// Version 1.2: add an index and a new preference
if (version_compare($oldversion, '1.2', '&lt;')) {
    $sqlarray = $dict->CreateIndexSQL(
        'idx_holidays_cat',
        CMS_DB_PREFIX . 'mod_holidays',
        'category'
    );
    $dict->ExecuteSQLArray($sqlarray);
    $this->SetPreference('default_category', '');
}

// Version 2.0: create a new related table
if (version_compare($oldversion, '2.0', '&lt;')) {
    $flds = "
        id I KEY AUTO,
        holiday_id I NOTNULL,
        tag C(100) NOTNULL
    ";
    $sqlarray = $dict->CreateTableSQL(CMS_DB_PREFIX . 'mod_holidays_tags', $flds);
    $dict->ExecuteSQLArray($sqlarray);
    $this->CreateEvent('HolidayTagged');
}

// Version 2.1: migrate data
if (version_compare($oldversion, '2.1', '&lt;')) {
    // Move category strings into the new tags table
    $rows = $db->GetAll('SELECT id, category FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE category != ?', ['']);
    foreach ($rows as $row) {
        $db->Execute(
            'INSERT INTO ' . CMS_DB_PREFIX . 'mod_holidays_tags (holiday_id, tag) VALUES (?, ?)',
            [$row['id'], $row['category']]
        );
    }
}
```

#### Key rules:

- Never assume the upgrade is from the immediately previous version — a user might jump from 1.0 to 2.1.
- Each `version_compare()` block should be independent and idempotent where possible.
- Test upgrades from multiple starting versions.

### The CmsDbQueryBase Class

For modules that display lists of items with pagination, CMSMS provides the abstract `CmsDbQueryBase` class. It handles query execution, pagination, and conversion of database rows into PHP objects.

#### Creating a query class

Extend `CmsDbQueryBase` and implement the two abstract methods — `execute()` and `GetObject()`:

```php
&lt;?php
// lib/class.HolidayQuery.php

class HolidayQuery extends CmsDbQueryBase
{
    public function execute()
    {
        if (!is_null($this->_rs)) return; // Don't execute twice

        $sql = 'SELECT SQL_CALC_FOUND_ROWS H.*
                FROM ' . CMS_DB_PREFIX . 'mod_holidays H';

        // Handle filtering
        $where = [];
        if (isset($this->_args['published'])) {
            $where[] = 'published = ' . (int) $this->_args['published'];
        }
        if (!empty($where)) {
            $sql .= ' WHERE ' . implode(' AND ', $where);
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

#### Using the query class

```
// In your action file
$query = new HolidayQuery(['published' => 1]);
$holidays = $query->GetMatches();
// Returns: array of HolidayItem objects

// Pagination info
$total = $query->TotalMatches();   // Total rows matching (ignoring limit)
$count = $query->RecordCount();    // Rows in current page
$pages = $query->numpages;         // Total number of pages
```

#### Setting the page limit

```php
// In the query class constructor
public function __construct($args = '')
{
    parent::__construct($args);
    if (isset($this->_args['limit'])) {
        $this->_limit = (int) $this->_args['limit'];
    }
}

// Usage
$query = new HolidayQuery(['published' => 1, 'limit' => 20]);
```

#### Manual iteration

Instead of `GetMatches()`, you can iterate through results manually:

```
$query = new HolidayQuery(['published' => 1]);
$query->execute();

while (!$query->EOF) {
    $holiday = $query->GetObject();
    // process $holiday...
    $query->MoveNext();
}
```

### CmsDbQueryBase API Reference

| Method / Property | Description |
| --- | --- |
| `execute()` | Abstract — build and execute the SQL query, populate `$_rs` and `$_totalmatchingrows` |
| `GetObject()` | Abstract — convert the current row (`$this->fields`) into a PHP object |
| `GetMatches()` | Execute and return all results as an array of objects |
| `TotalMatches()` | Total rows matching the query (ignoring limit/offset) |
| `RecordCount()` | Number of rows in the current page |
| `MoveNext()` | Advance to the next row |
| `MoveFirst() / Rewind()` | Reset to the first row |
| `EOF()` | True if past the last row |
| `Close()` | Free the result set resources |
| `$_args` (protected) | Arguments passed to the constructor |
| `$_limit` (protected) | Page limit (default: 1000) |
| `$_offset` (protected) | Row offset (default: 0) |
| `numpages` (read-only) | Total number of pages |
| `fields` (read-only) | Associative array of the current row |

### Next Steps

This completes the Database Operations chapter. Continue to [Templates](/modules/templates/) to learn how to use Smarty templates for admin and frontend output.
