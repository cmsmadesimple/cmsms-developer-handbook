## Tag Parameters

When your module is called from a Smarty template, parameters can be passed to it. On the frontend, these parameters must be registered for security. On the admin side, you can document them for the module help page.

### Frontend Parameter Registration

For security, CMSMS strips any frontend parameters that are not explicitly registered. Register parameters in `InitializeFrontend()` using `SetParameterType()`:

```php
protected function InitializeFrontend()
{
    $this->RegisterModulePlugin();
    $this->SetParameterType('hid', CLEAN_INT);
    $this->SetParameterType('page', CLEAN_INT);
    $this->SetParameterType('limit', CLEAN_INT);
    $this->SetParameterType('detailpage', CLEAN_STRING);
    $this->SetParameterType('detailtemplate', CLEAN_STRING);
    $this->SetParameterType('category', CLEAN_STRING);
}
```

Parameters only need to be registered once — they apply to all frontend actions.

#### Cleaning constants

| Constant | Effect | Use for |
| --- | --- | --- |
| `CLEAN_INT` | Casts to integer | IDs, page numbers, limits, numeric flags |
| `CLEAN_FLOAT` | Casts to float | Prices, coordinates, percentages |
| `CLEAN_STRING` | Strips tags and trims whitespace | Page aliases, template names, categories, slugs |
| `CLEAN_NONE` | No cleaning — passed through raw | Use with extreme caution. Only when you need raw HTML input and will sanitize it yourself. |

> **Note:** Unregistered parameters are silently dropped from the `$params` array in frontend actions. This is a security feature — it prevents attackers from injecting unexpected parameters. Always register every parameter your frontend actions expect.

### Admin Parameter Registration

Admin action parameters are not filtered through `SetParameterType()` — all submitted parameters appear in the `$params` array. However, you should still sanitize them manually in your action code (see [Securing Input](/modules/security/securing-input/)).

### Documenting Parameters

Use `CreateParameter()` in `InitializeAdmin()` to document your module's tag parameters. These appear in the module help page (accessible from Extensions > Tags or the Module Manager):

```php
protected function InitializeAdmin()
{
    $this->CreateParameter('hid', null, $this->Lang('param_hid'));
    $this->CreateParameter('limit', 1000, $this->Lang('param_limit'));
    $this->CreateParameter('detailpage', null, $this->Lang('param_detailpage'));
    $this->CreateParameter('detailtemplate', null, $this->Lang('param_detailtemplate'));
    $this->CreateParameter('category', null, $this->Lang('param_category'));
}
```

#### CreateParameter() arguments

| Argument | Description |
| --- | --- |
| `param` | The parameter name (must match what you registered with SetParameterType) |
| `defaultval` | The default value (displayed in help) |
| `helpstring` | A description of the parameter (use a Lang string) |

And in your `lang/en_US.php`:

```
$lang['param_hid'] = 'Applicable to the detail action. The integer ID of the holiday to display.';
$lang['param_limit'] = 'Applicable to the default action. Limits the number of holidays displayed. Default: 1000.';
$lang['param_detailpage'] = 'Applicable to the default action. The page alias or ID on which to display detail views.';
$lang['param_detailtemplate'] = 'Applicable to the detail action. An alternate template file to use for the detail view.';
$lang['param_category'] = 'Filter holidays by category name.';
```

### How Parameters Flow

Here is the complete flow of a parameter from the template call to your action:

```
{* 1. Content editor places the tag with parameters *}
{Holidays limit=10 detailpage=holiday-detail}

{* 2. CMSMS checks each parameter against SetParameterType registrations *}
{*    - 'limit' is registered as CLEAN_INT → cast to integer 10 *}
{*    - 'detailpage' is registered as CLEAN_STRING → trimmed and stripped *}
{*    - Any unregistered parameters are silently dropped *}

{* 3. The cleaned parameters arrive in $params in your action file *}
```
```
// action.default.php
$limit = isset($params['limit']) ? (int) $params['limit'] : 1000;
$detailpage = isset($params['detailpage']) ? $params['detailpage'] : $returnid;
```

### Restricting Unknown Parameters

By default, unregistered frontend parameters are silently dropped. You can optionally generate an error when unknown parameters are passed by calling `RestrictUnknownParams()`:

```php
protected function InitializeFrontend()
{
    $this->RegisterModulePlugin();
    $this->RestrictUnknownParams(true);
    $this->SetParameterType('hid', CLEAN_INT);
    // ...
}
```

This is useful during development to catch typos in template calls, but should generally be left off in production to avoid breaking pages if a parameter name changes.

### Complete Example

```php
// Holidays.module.php

protected function InitializeFrontend()
{
    $this->RegisterModulePlugin();
    $this->SetParameterType('hid', CLEAN_INT);
    $this->SetParameterType('limit', CLEAN_INT);
    $this->SetParameterType('detailpage', CLEAN_STRING);
    $this->SetParameterType('detailtemplate', CLEAN_STRING);
}

protected function InitializeAdmin()
{
    $this->CreateParameter('hid', null,
        $this->Lang('param_hid'));
    $this->CreateParameter('limit', 1000,
        $this->Lang('param_limit'));
    $this->CreateParameter('detailpage', null,
        $this->Lang('param_detailpage'));
    $this->CreateParameter('detailtemplate', null,
        $this->Lang('param_detailtemplate'));
}
```

### Next Steps

This completes the Smarty Tags chapter. Continue to [Module Preferences](/modules/preferences/) to learn how to store and retrieve module settings.
