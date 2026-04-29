## Creating Tables

CMSMS provides the ADODB DataDictionary class for creating, altering, and dropping database tables in a database-agnostic way. While CMSMS only supports MySQL, using the DataDictionary keeps your code consistent with the rest of the ecosystem and handles table prefixing and SQL generation for you.

### Creating a Table

```php
// method.install.php
if (!defined('CMS_VERSION')) exit;

$db = $this->GetDb();
$dict = NewDataDictionary($db);
$taboptarray = array('mysql' => 'TYPE=MyISAM');

$flds = "
    id I KEY AUTO,
    name C(255) KEY NOTNULL,
    description X,
    published I1,
    the_date I NOTNULL,
    created I,
    modified I
";

$sqlarray = $dict->CreateTableSQL(CMS_DB_PREFIX . 'mod_holidays', $flds, $taboptarray);
$dict->ExecuteSQLArray($sqlarray);
```

### DataDictionary Field Types

| Type code | MySQL equivalent | Description |
| --- | --- | --- |
| `C(n)` | VARCHAR(n) | Character field with max length n |
| `C2(n)` | VARCHAR(n) (unicode) | Unicode character field |
| `X` | TEXT | Large text field |
| `X2` | LONGTEXT | Very large text field |
| `I` | INT | Integer |
| `I1` | TINYINT | Small integer (0-255, good for booleans/flags) |
| `I2` | SMALLINT | Medium integer |
| `I4` | INT | Standard integer (same as I) |
| `I8` | BIGINT | Large integer |
| `F` | DOUBLE | Floating point number |
| `N` | DECIMAL | Fixed-point number |
| `D` | DATE | Date field |
| `T` | DATETIME | Date and time field |
| `L` | TINYINT(1) | Boolean / logical field |
| `B` | BLOB | Binary large object |

### Field Modifiers

| Modifier | Description |
| --- | --- |
| `KEY` | Part of the primary key |
| `AUTO` | Auto-increment (use with I KEY) |
| `NOTNULL` | NOT NULL constraint |
| `DEFAULT 'value'` | Default value |
| `UNSIGNED` | Unsigned integer |

### Dropping a Table

```php
// method.uninstall.php
$db = $this->GetDb();
$dict = NewDataDictionary($db);
$sqlarray = $dict->DropTableSQL(CMS_DB_PREFIX . 'mod_holidays');
$dict->ExecuteSQLArray($sqlarray);
```

### Adding Columns

```php
// method.upgrade.php
$db = $this->GetDb();
$dict = NewDataDictionary($db);
$sqlarray = $dict->AddColumnSQL(
    CMS_DB_PREFIX . 'mod_holidays',
    'category C(100), sort_order I DEFAULT 0'
);
$dict->ExecuteSQLArray($sqlarray);
```

### Adding Indexes

```
$sqlarray = $dict->CreateIndexSQL(
    'idx_holidays_date',                    // index name
    CMS_DB_PREFIX . 'mod_holidays',         // table name
    'the_date'                              // column(s)
);
$dict->ExecuteSQLArray($sqlarray);

// Unique index
$sqlarray = $dict->CreateIndexSQL(
    'idx_holidays_name',
    CMS_DB_PREFIX . 'mod_holidays',
    'name',
    array('UNIQUE')
);
$dict->ExecuteSQLArray($sqlarray);
```

### Table Naming Convention

Always prefix your table names with `CMS_DB_PREFIX` followed by `mod_` and your module's short name:

```
CMS_DB_PREFIX . 'mod_holidays'       // Main table
CMS_DB_PREFIX . 'mod_holidays_cats'  // Related table
```

### Next Steps

Continue to [Schema Management](/modules/database/schema-management/) to learn how to handle schema migrations across versions and build reusable query classes.
