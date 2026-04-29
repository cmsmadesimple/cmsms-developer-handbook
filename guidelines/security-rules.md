## Security Guidelines

These rules detect security vulnerabilities in your module code — SQL injection, XSS, file inclusion, command injection, CSRF, hardcoded credentials, and other attack vectors. Every high-severity finding here represents a real exploitable vulnerability.

These guidelines are enforced by [Module Check](/module-check) and the [CMSMS Scanner](/cmsms-scanner). Each rule includes a severity level, explanation, and code examples.

### CMSMS\_SEC\_001: User input flows into SQL query

**Severity:** high

User-controlled values concatenated into SQL strings allow injection. The ADODB layer in CMSMS supports parameterized queries natively.

**Fix:** Use ? placeholders and pass values as the second array argument to the ADODB method.

```sql
// Bad
$db-&gt;GetRow('SELECT * FROM ' . CMS_DB_PREFIX . 'items WHERE id = ' . $params['id']);
```
```sql
// Good
$db-&gt;GetRow('SELECT * FROM ' . CMS_DB_PREFIX . 'items WHERE id = ?', [(int)$params['id']]);
```

### CMSMS\_SEC\_002: User input flows into echo/print output

**Severity:** high

Echoing user input without htmlspecialchars() allows attackers to inject script tags. This is the most common XSS pattern in CMSMS modules.

**Fix:** Wrap in htmlspecialchars($value, ENT\_QUOTES, 'UTF-8') or pass to a Smarty template with |escape.

```
// Bad
echo 'Search: ' . $params['query'];
```
```
// Good
echo 'Search: ' . htmlspecialchars($params['query'] ?? '', ENT_QUOTES, 'UTF-8');
```

### CMSMS\_SEC\_003: User input flows into file path operation

**Severity:** high

User-controlled path segments enable directory traversal (../) to read/write/delete arbitrary files on the server.

**Fix:** Use basename() to strip path components, or realpath() + str\_starts\_with() to verify the resolved path stays within the allowed directory.

```
// Bad
$content = file_get_contents($upload_dir . '/' . $params['filename']);
```
```
// Good
$safe = basename($params['filename'] ?? '');
$path = realpath($upload_dir . '/' . $safe);
if ($path &amp;&amp; str_starts_with($path, realpath($upload_dir))) {
    $content = file_get_contents($path);
}
```

### CMSMS\_SEC\_004: User input flows into include/require

**Severity:** high

Including files based on user input allows attackers to execute arbitrary PHP via LFI (local file inclusion) or RFI (remote file inclusion).

**Fix:** Use an allowlist of valid include targets. Never pass user input to include/require.

```
// Bad
include($module_path . '/views/' . $params['view'] . '.php');
```
```
// Good
$allowed = ['list', 'detail', 'edit'];
$view = in_array($params['view'] ?? '', $allowed, true) ? $params['view'] : 'list';
include($module_path . '/views/' . $view . '.php');
```

### CMSMS\_SEC\_005: User input flows into shell command

**Severity:** high

User input in shell commands enables OS command injection. Attackers can chain commands with ;, |, &&, or backticks.

**Fix:** Avoid shell commands entirely. If unavoidable, use escapeshellarg() on every argument.

```
// Bad
exec('convert ' . $params['file'] . ' output.png');
```
```
// Good
exec('convert ' . escapeshellarg($validated_path) . ' ' . escapeshellarg($output));
```

### CMSMS\_SEC\_006: SQL query with concatenated variable after WHERE/SET/VALUES

**Severity:** high

Variables concatenated into WHERE/SET/VALUES clauses are the primary SQL injection vector. CMS\_DB\_PREFIX concatenation in FROM clauses is safe and excluded.

**Fix:** Replace the concatenated variable with a ? placeholder and add the value to the parameter array.

```
// Bad
$db-&gt;Execute('UPDATE ' . CMS_DB_PREFIX . 'items SET name = \'' . $name . '\' WHERE id = ' . $id);
```
```
// Good
$db-&gt;Execute('UPDATE ' . CMS_DB_PREFIX . 'items SET name = ? WHERE id = ?', [$name, $id]);
```

### CMSMS\_SEC\_007: SQL query with PHP variable interpolation in double-quoted string

**Severity:** high

PHP interpolates variables inside double-quoted strings before the query reaches the database. This is functionally identical to concatenation-based injection.

**Fix:** Switch to single-quoted strings with ? placeholders. Concatenate only CMS\_DB\_PREFIX.

```
// Bad
$db-&gt;Execute("UPDATE {$prefix}items SET status = $status WHERE id = $id");
```
```
// Good
$db-&gt;Execute('UPDATE ' . CMS_DB_PREFIX . 'items SET status = ? WHERE id = ?', [$status, $id]);
```

### CMSMS\_SEC\_008: LIKE clause with concatenated search term

**Severity:** medium

LIKE clauses with concatenated variables are injectable. The % wildcards should be prepended/appended to the bound parameter value, not embedded in the SQL string.

**Fix:** Use LIKE ? and pass '%' . $search . '%' as the parameter value.

```sql
// Bad
$db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'items WHERE name LIKE \'%' . $search . '%\'');
```
```sql
// Good
$db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'items WHERE name LIKE ?', ['%' . $search . '%']);
```

### CMSMS\_SEC\_009: Dynamic ORDER BY or table name from user input

**Severity:** high

ORDER BY columns and table names cannot be parameterized with ?. User input in these positions must be validated against an allowlist.

**Fix:** Validate against an explicit allowlist of permitted column/table names.

```sql
// Bad
$db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'items ORDER BY ' . $params['sort']);
```
```sql
// Good
$allowed_cols = ['name', 'created', 'id'];
$sort = in_array($params['sort'] ?? '', $allowed_cols, true) ? $params['sort'] : 'id';
$db-&gt;GetAll('SELECT * FROM ' . CMS_DB_PREFIX . 'items ORDER BY ' . $sort);
```

### CMSMS\_SEC\_010: eval() with variable argument

**Severity:** high

eval() executes its string argument as PHP. Any variable content leads directly to Remote Code Execution.

**Fix:** Eliminate eval(). Use callbacks, match expressions, or configuration arrays.

```
// Bad
eval('$result = ' . $params['formula'] . ';');
```
```
// Good
$formulas = ['sum' =&gt; fn($a, $b) =&gt; $a + $b];
$result = ($formulas[$params['formula']] ?? fn() =&gt; 0)($a, $b);
```

### CMSMS\_SEC\_011: unserialize() on external or variable data

**Severity:** high

unserialize() instantiates objects and triggers magic methods (\_\_wakeup, \_\_destruct). With user-controlled data, attackers chain gadgets for RCE.

**Fix:** Use json\_decode() for data interchange. If unserialize() is required, restrict: unserialize($data, ['allowed\_classes' => false]).

```
// Bad
$settings = unserialize($params['config']);
```
```
// Good
$settings = json_decode($params['config'] ?? '{}', true, 512, JSON_THROW_ON_ERROR);
```

### CMSMS\_SEC\_012: File upload with user-controlled destination filename

**Severity:** high

The original filename from $\_FILES is fully user-controlled. It can contain ../ for traversal or .php extensions to upload executable code.

**Fix:** Generate a safe filename server-side. Validate extension against an allowlist.

```
// Bad
move_uploaded_file($_FILES['doc']['tmp_name'], $dir . '/' . $_FILES['doc']['name']);
```
```
// Good
$ext = strtolower(pathinfo($_FILES['doc']['name'], PATHINFO_EXTENSION));
if (!in_array($ext, ['pdf', 'doc', 'txt'], true)) { return; }
$safe = bin2hex(random_bytes(16)) . '.' . $ext;
move_uploaded_file($_FILES['doc']['tmp_name'], $dir . '/' . $safe);
```

### CMSMS\_SEC\_013: Frontend form submission handler without CSRF validation

**Severity:** medium

Frontend state-changing operations triggered by form submission without CSRF token validation are vulnerable to Cross-Site Request Forgery. Admin actions are excluded because CMSMS handles admin CSRF protection at the framework level. For frontend forms, the CMSMSExt module offers a robust CSRF solution.

**Fix:** Install the CMSMSExt module. In templates, add {xt\_form\_csrf} inside your form: {form\_start}{xt\_form\_csrf}. In the action handler, validate with: if(!\xt\_utils::valid\_form\_csrf()) { throw new Exception($this->Lang('error\_security')); }. This is recommended best practice but not mandatory.

```
// Bad
if (isset($params['submit'])) {
    $db-&gt;Execute('DELETE FROM items WHERE id = ?', [$params['id']]);
}
```
```php
// Good
{* In template: *}
{form_start}{xt_form_csrf}
...
{form_end}

// In action handler:
if(!\xt_utils::valid_form_csrf()) {
    throw new Exception($this-&gt;Lang('error_security'));
}
$db-&gt;Execute('DELETE FROM items WHERE id = ?', [(int)$params['id']]);
```

### CMSMS\_SEC\_014: Hardcoded credential assigned to variable

**Severity:** high

Hardcoded secrets in source code are exposed in version control and cannot be rotated without deployment.

**Fix:** Store in CMSMS site preferences via cms\_siteprefs::get(), environment variables, or config.php.

```
// Bad
$api_key = 'sk_live_EXAMPLE_REPLACE_WITH_REAL_KEY';
```
```
// Good
$api_key = cms_siteprefs::get('mymodule_api_key', '');
```

### CMSMS\_SEC\_015: User input decoded from base64 then used in dangerous sink

**Severity:** high

Base64-decoding user input and passing the result to a dangerous sink bypasses WAFs and input filters.

**Fix:** Validate and sanitize the decoded content before use.

```
// Bad
$html = base64_decode($_POST['content']);
echo $html;
```
```
// Good
$raw = base64_decode($params['content'] ?? '', true);
if ($raw === false) { return; }
echo htmlspecialchars($raw, ENT_QUOTES, 'UTF-8');
```

### CMSMS\_SEC\_016: preg\_replace with /e modifier on user input

**Severity:** high

The /e modifier in preg\_replace() evaluates the replacement string as PHP code. With user-controlled input, this is RCE.

**Fix:** Replace with preg\_replace\_callback() which uses a proper callable instead of eval.

```
// Bad
preg_replace('/\{(\w+)\}/e', '$data["$1"]', $params['template']);
```
```
// Good
preg_replace_callback('/\{(\w+)\}/', fn($m) =&gt; htmlspecialchars($data[$m[1]] ?? ''), $params['template']);
```

### CMSMS\_SEC\_017: Header injection via user input in header()

**Severity:** high

User input in HTTP headers enables response splitting or open redirects.

**Fix:** Validate the URL against an allowlist. Use CMSMS's $this->RedirectToAdminTab() for admin redirects.

```
// Bad
header('Location: ' . $_GET['return_url']);
```
```php
// Good
$this-&gt;RedirectToAdminTab($id);
```

### CMSMS\_SEC\_018: Code obfuscation tool detected

**Severity:** high

Obfuscated/encoded PHP code hides its true functionality, making security review impossible. Modules distributed for CMSMS must have readable, reviewable source code. Zend Guard, SourceGuardian, and ionCube encoded files are not permitted.

**Fix:** Distribute the original unencoded PHP source code. If you need to protect intellectual property, use a license key system instead of code obfuscation.

```php
// Bad
&lt;?php @Zend; // encoded file
```
```php
// Good
&lt;?php
// Readable, reviewable source code
```
