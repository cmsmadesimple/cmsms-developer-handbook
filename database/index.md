## Database Operations

CMSMS uses a modified version of adodb-lite for database abstraction. While only MySQL is supported, the abstraction layer provides parameterized queries, a DataDictionary for schema management, and a query base class for building reusable result sets.

### In This Chapter

- [Using ADODB](/modules/database/using-adodb/) — Getting the database connection, executing queries, fetching results, and using parameterized queries.
- [Creating Tables](/modules/database/creating-tables/) — Using the DataDictionary to create, alter, and drop tables in a database-agnostic way.
- [Schema Management](/modules/database/schema-management/) — Managing schema changes across module versions using method.upgrade.php, and building query classes with CmsDbQueryBase.
