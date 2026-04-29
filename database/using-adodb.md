## Using ADODB

CMSMS creates a single database connection per request. You access it through the module's `GetDb()` method or the static `\cms_utils::get_db()` utility. All database operations go through this connection object, which provides methods for executing queries, fetching results, and handling errors.

### Getting the Connection

```php
// From within a module action or lifecycle file
$db = $this->GetDb();

// From within a model class or anywhere else
$db = \cms_utils::get_db();
```

Both return the same single connection instance.

### Executing Queries

#### Execute() — INSERT, UPDATE, DELETE

```sql
// Insert
$sql = 'INSERT INTO ' . CMS_DB_PREFIX . 'mod_holidays (name, the_date, published) VALUES (?, ?, ?)';
$db->Execute($sql, [$name, $the_date, 1]);

// Get the auto-increment ID of the last insert
$new_id = $db->Insert_ID();

// Update
$sql = 'UPDATE ' . CMS_DB_PREFIX . 'mod_holidays SET name = ?, published = ? WHERE id = ?';
$db->Execute($sql, [$name, $published, $id]);

// Delete
$sql = 'DELETE FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE id = ?';
$db->Execute($sql, [$id]);
```

#### GetRow() — fetch a single row

```sql
$sql = 'SELECT * FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE id = ?';
$row = $db->GetRow($sql, [$id]);
// Returns: ['id' => 1, 'name' => 'Christmas', ...] or false
```

#### GetOne() — fetch a single value

```sql
$sql = 'SELECT COUNT(*) FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE published = 1';
$count = $db->GetOne($sql);
// Returns: '42' (string) or false
```

#### GetAll() — fetch all rows

```sql
$sql = 'SELECT * FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE published = 1 ORDER BY the_date DESC';
$rows = $db->GetAll($sql);
// Returns: array of associative arrays, or empty array
```

#### GetCol() — fetch a single column

```sql
$sql = 'SELECT name FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE published = 1';
$names = $db->GetCol($sql);
// Returns: ['Christmas', 'New Year', ...] or empty array
```

#### SelectLimit() — fetch with pagination

```sql
$sql = 'SELECT * FROM ' . CMS_DB_PREFIX . 'mod_holidays ORDER BY the_date DESC';
$rs = $db->SelectLimit($sql, $limit, $offset);
// $rs is a resultset object — iterate with while (!$rs->EOF) { ... $rs->MoveNext(); }
```

### Parameterized Queries

Always use `?` placeholders and pass values as an array. The database layer quotes and escapes values based on their PHP data type:

```sql
// SAFE — parameterized
$db->Execute('SELECT * FROM ' . CMS_DB_PREFIX . 'mod_holidays WHERE id = ?', [$id]);
$db->Execute('INSERT INTO ' . CMS_DB_PREFIX . 'mod_holidays (name, the_date) VALUES (?, ?)',
             [$name, $the_date]);

// DANGEROUS — never do this
$db->Execute("SELECT * FROM " . CMS_DB_PREFIX . "mod_holidays WHERE id = $id");
$db->Execute("SELECT * FROM " . CMS_DB_PREFIX . "mod_holidays WHERE name = '$name'");
```

### Error Handling

```
$dbr = $db->Execute($sql, $params);
if (!$dbr) {
    // Query failed
    $error = $db->ErrorMsg();
    throw new \CmsSQLErrorException($db->sql . ' -- ' . $error);
}
```

### Working with Result Sets

Methods like `SelectLimit()` return a result set object that you iterate through:

```sql
$sql = 'SELECT * FROM ' . CMS_DB_PREFIX . 'mod_holidays ORDER BY the_date DESC';
$rs = $db->SelectLimit($sql, 20, 0);

while ($rs && !$rs->EOF) {
    $name = $rs->fields['name'];
    $date = $rs->fields['the_date'];
    // process row...
    $rs->MoveNext();
}
if ($rs) $rs->Close();
```

### Table Prefix

Always use `CMS_DB_PREFIX` before your table names. This constant contains the table prefix configured during CMSMS installation (typically `cms_`):

```
$table = CMS_DB_PREFIX . 'mod_holidays';
// Produces: cms_mod_holidays
```

### Common Methods Reference

| Method | Returns | Use for |
| --- | --- | --- |
| `Execute($sql, $params)` | Result set or false | INSERT, UPDATE, DELETE, and general queries |
| `GetRow($sql, $params)` | Associative array or false | Fetching a single row |
| `GetOne($sql, $params)` | Scalar value or false | Fetching a single value (COUNT, MAX, etc.) |
| `GetAll($sql, $params)` | Array of arrays | Fetching all matching rows |
| `GetCol($sql, $params)` | Array of values | Fetching a single column |
| `SelectLimit($sql, $limit, $offset)` | Result set object | Paginated queries |
| `Insert_ID()` | Integer | Last auto-increment ID after INSERT |
| `ErrorMsg()` | String | Last error message |

### Next Steps

Continue to [Creating Tables](/modules/database/creating-tables/) to learn how to use the DataDictionary for schema management.
