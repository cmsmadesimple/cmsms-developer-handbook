## Code Quality Guidelines

These rules flag code quality issues that affect maintainability, performance, and reliability. While not security vulnerabilities, they indicate patterns that lead to bugs, poor performance, or difficulty maintaining the module over time.

These guidelines are enforced by [Module Check](/module-check) and the [CMSMS Scanner](/cmsms-scanner). Each rule includes a severity level, explanation, and code examples.

### CMSMS\_QUAL\_001: Inline <script> block in PHP echo output

**Severity:** low

Inline JS in PHP output prevents browser caching, blocks CSP enforcement, and tangles presentation with server logic.

**Fix:** Move JS to a .js file in assets/ and load via $this->AddHeaderJavascript() or a template <script src>.

```
// Bad
echo '&lt;script&gt;document.getElementById("btn").addEventListener("click", function() { submit(); });&lt;/script&gt;';
```
```php
// Good
$this-&gt;AddHeaderJavascript($this-&gt;GetModuleURLPath() . '/assets/admin.js');
```

### CMSMS\_QUAL\_002: Large inline <style> block in template

**Severity:** low

Large inline CSS blocks prevent browser caching, increase page weight on every request, and make styling inconsistent across module views.

**Fix:** Move to a .css file in assets/ and load via $this->AddHeaderCSS() or a <link> tag.

```
// Bad
&lt;style&gt;
.mymod-table { width: 100%; border-collapse: collapse; }
.mymod-table th { background: #f5f5f5; padding: 8px; }
.mymod-table td { padding: 8px; border-bottom: 1px solid #ddd; }
&lt;/style&gt;
```
```
// Good
{* In assets/admin.css *}
&lt;link rel="stylesheet" href="{$module_url}/assets/admin.css"&gt;
```

### CMSMS\_QUAL\_003: Database query inside a loop body

**Severity:** medium

Each loop iteration fires a separate DB round-trip. For 100 items, that's 100 queries instead of 1. This is the classic N+1 problem.

**Fix:** Fetch all needed data in a single query before the loop using IN (...) or a JOIN.

```sql
// Bad
foreach ($category_ids as $cid) {
    $items = $db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'module_mymod_items WHERE cat_id = ?', [$cid]);
}
```
```sql
// Good
$ph = implode(',', array_fill(0, count($category_ids), '?'));
$items = $db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . "module_mymod_items WHERE cat_id IN ($ph)", $category_ids);
```

### CMSMS\_QUAL\_004: Database access in Smarty template

**Severity:** high

Templates are the view layer. Database access in templates makes debugging impossible, creates security risks, and prevents template caching.

**Fix:** Move all queries to the action file. Pass results to the template via $smarty->assign().

```sql
// Bad
{php}
$db = cmsms()-&gt;GetDb();
$rows = $db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'items');
{/php}
```
```smarty
// Good
{foreach $items as $item}
  &lt;li&gt;{$item.name|escape}&lt;/li&gt;
{/foreach}
```

### CMSMS\_QUAL\_005: Error suppression operator on I/O or parsing function

**Severity:** medium

@ silences errors, making failures invisible. A failed unlink or fopen produces no feedback, leading to silent data loss or broken state.

**Fix:** Check preconditions (is\_file, is\_writable) or use try/catch. Handle the error explicitly.

```
// Bad
@unlink($cache_file);
$data = @file_get_contents($url);
```
```
// Good
if (is_file($cache_file)) { unlink($cache_file); }
$data = file_get_contents($url);
if ($data === false) { /* handle error */ }
```

### CMSMS\_QUAL\_006: Action file mixes DB queries with HTML echo

**Severity:** medium

Mixing data access and HTML rendering in one file makes both untestable, prevents template overrides, and creates maintenance burden.

**Fix:** Keep the action file for data logic only. Assign results to $smarty and render via a .tpl template.

```sql
// Bad
$rows = $db-&gt;GetAll('SELECT * FROM items');
echo '&lt;table&gt;';
foreach ($rows as $r) { echo '&lt;tr&gt;&lt;td&gt;' . $r['name'] . '&lt;/td&gt;&lt;/tr&gt;'; }
echo '&lt;/table&gt;';
```
```sql
// Good
$rows = $db-&gt;GetAll('SELECT * FROM items');
$tpl-&gt;assign('items', $rows);
echo $tpl-&gt;fetch();
```

### CMSMS\_QUAL\_007: Deeply nested control structures (4+ levels)

**Severity:** medium

Deep nesting makes code hard to follow, test, and modify. Each level multiplies the number of execution paths.

**Fix:** Use early returns/continue (guard clauses), extract inner blocks into named methods, or use array\_filter/array\_map.

```
// Bad
foreach ($mods as $m) {
    if ($m-&gt;isActive()) {
        foreach ($m-&gt;getItems() as $i) {
            if ($i-&gt;isValid()) {
                if ($i-&gt;hasAccess($u)) {
                    process($i);
                }
            }
        }
    }
}
```
```php
// Good
foreach ($mods as $m) {
    if (!$m-&gt;isActive()) continue;
    $this-&gt;processItems($m, $u);
}
```

### CMSMS\_QUAL\_008: Overly large file (500+ lines)

**Severity:** low

Large files are hard to navigate, review, and test. In CMSMS modules, action files and the main module class tend to accumulate logic that belongs in lib/ classes.

**Fix:** Extract cohesive functionality into focused classes under lib/. Use the module autoloader.

```
// Bad
// action.defaultadmin.php — 800 lines handling list, edit, delete, settings, export
```
```
// Good
// action.defaultadmin.php — 30 lines delegating to:
$controller = new MyModule\AdminController($this, $smarty);
$controller-&gt;dispatch($params);
```

### CMSMS\_QUAL\_009: SELECT \* in production query

**Severity:** low

SELECT \* fetches all columns including BLOBs and unused fields, wastes memory, and makes code fragile when columns are added/removed.

**Fix:** List only the columns you need: SELECT id, name, status FROM ...

```sql
// Bad
$rows = $db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'module_mymod_items');
```
```sql
// Good
$rows = $db-&gt;GetAll('SELECT id, name, status FROM ' . CMS_DB_PREFIX . 'module_mymod_items');
```

### CMSMS\_QUAL\_010: Catch block that silently swallows exception

**Severity:** medium

Empty catch blocks hide failures. The code continues in an unexpected state, making bugs extremely hard to diagnose.

**Fix:** At minimum, log the exception. Better: handle it or let it propagate.

```
// Bad
try {
    $result = $db-&gt;Execute($query, $params);
} catch (Exception $e) {
    // ignore
}
```
```php
// Good
try {
    $result = $db-&gt;Execute($query, $params);
} catch (Exception $e) {
    audit($this-&gt;GetName() . ': DB error: ' . $e-&gt;getMessage());
    $this-&gt;SetError($this-&gt;Lang('error_db_operation'));
}
```

### CMSMS\_QUAL\_011: Deprecated PHP function used

**Severity:** medium

These functions are deprecated or removed in PHP 7.x/8.x. split() was removed in PHP 7.0 (use explode or preg\_split). ereg functions were removed in PHP 7.0 (use preg\_match/preg\_replace). mysql\_\* was removed in PHP 7.0 (use CMSMS database API). each() was deprecated in PHP 7.2. create\_function() was deprecated in PHP 7.2 (use closures). mcrypt\_\* was removed in PHP 7.2 (use openssl).

**Fix:** Replace with the modern equivalent: split() -> explode()/preg\_split(), ereg -> preg\_match(), mysql\_\* -> cms\_utils::get\_db(), each() -> foreach, create\_function() -> fn() or closure, mcrypt\_\* -> openssl\_\*.

```sql
// Bad
$parts = split(',', $str);
if (ereg('^[a-z]+$', $input)) { ... }
mysql_query('SELECT * FROM items');
```
```sql
// Good
$parts = explode(',', $str);
if (preg_match('/^[a-z]+$/', $input)) { ... }
$db = cms_utils::get_db();
$db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'items');
```

### CMSMS\_QUAL\_012: Localhost or loopback URL hardcoded

**Severity:** medium

Hardcoded localhost URLs are development leftovers that will fail on any other environment. They may also leak internal service endpoints or development infrastructure details.

**Fix:** Remove hardcoded localhost URLs. Use CMSMS configuration values, site preferences, or relative URLs instead.

```
// Bad
$api = 'http://localhost:8080/api/check';
```
```php
// Good
$api = $this-&gt;GetPreference('mymodule_api_url', '');
```

### CMSMS\_QUAL\_013: Form submit handler without POST-Redirect-Get pattern

**Severity:** medium

When a user submits a form and the action processes it without redirecting, refreshing the page will re-submit the form. The PRG (Post-Redirect-Get) pattern prevents this by redirecting after processing. In CMSMS, use $this->RedirectToAdminTab() after handling POST data.

**Fix:** After processing the form submission, redirect: $this->SetMessage($this->Lang('item\_saved')); $this->RedirectToAdminTab();

```sql
// Bad
if (isset($params['submit'])) {
    $db-&gt;Execute('INSERT INTO ...', [...]);
    // no redirect - refresh will re-insert
}
```
```php
// Good
if (isset($params['submit'])) {
    $db-&gt;Execute('INSERT INTO ...', [...]);
    $this-&gt;SetMessage($this-&gt;Lang('item_saved'));
    $this-&gt;RedirectToAdminTab();
}
```

### CMSMS\_QUAL\_014: Disallowed file type in module directory

**Severity:** high

Production modules should not contain: compressed archives (.zip, .gz, .tar, .rar), binary executables (.exe, .bin, .phar, .dll, .so), version control directories (.git, .svn), or AI tool directories (.cursor, .claude, .aider). These bloat the module, pose security risks, or leak development metadata.

**Fix:** Remove disallowed files before distribution. Use .distignore or a build script to exclude development files from release packages.

```
// Bad
modules/MyModule/
├── .git/
├── backup.zip
├── tool.exe
└── .cursor/
```
```
// Good
modules/MyModule/
├── MyModule.module.php
├── lib/
├── templates/
└── assets/
```
