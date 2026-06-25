---
inclusion: fileMatch
fileMatchPattern: "**/*.md"
---

# Code Example Standards for CMSMS Documentation

## PHP Examples

### Action files must include the security header:
```php
if (!defined('CMS_VERSION')) exit;
if (!$this->CheckPermission(ModuleName::MANAGE_PERM)) return;
```

### Database queries must use parameterized queries:
```php
// CORRECT
$db->Execute('SELECT * FROM '.CMS_DB_PREFIX.'mod_table WHERE id=?', [$id]);

// NEVER show this pattern
$db->Execute("SELECT * FROM table WHERE id=$id");  // SQL injection!
```

### Template loading must use the standard pattern:
```php
$smarty = cmsms()->GetSmarty();
$tpl = $smarty->CreateTemplate($this->GetTemplateResource('mytemplate.tpl'), null, null, $smarty);
$tpl->assign('items', $items);
$tpl->display();
```

### Always use CMS_DB_PREFIX for table names:
```php
CMS_DB_PREFIX.'module_mymodule'  // Correct
'cms_module_mymodule'            // Never hardcode the prefix
```

## Smarty Template Examples

### Admin forms use CMSMS form plugins:
```smarty
{form_start action="save"}
{cms_help_title title="Field Name"}{cms_help_content}Help text{/cms_help_content}
<input type="text" name="{$actionid}name" value="{$item->name|cms_escape}">
{form_end}
```

### Always escape output in templates:
```smarty
{$variable|cms_escape}          {* HTML context *}
{$variable|cms_escape:'url'}    {* URL context *}
```

## Patterns to NEVER Show (Security Violations)
- `{php}` tags in templates
- Unparameterized SQL queries with user input
- `echo` or `print` for HTML output (use templates)
- `$_GET`, `$_POST`, `$_REQUEST` directly (use `$params`)
- `eval()`, `exec()`, `system()`, `passthru()`
- `include`/`require` with user-controlled paths
