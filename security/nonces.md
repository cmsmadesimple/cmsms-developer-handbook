## Nonces (CSRF Protection)

Cross-Site Request Forgery (CSRF) is an attack where a malicious website tricks an authenticated user's browser into submitting a request to your CMSMS site — deleting records, changing settings, or performing other actions without the user's knowledge.

CMSMS protects against CSRF through hidden tokens embedded in forms. When you use the built-in Smarty form plugins, this protection is handled automatically.

### How CSRF Attacks Work

Imagine an admin user is logged into CMSMS. While browsing another site, that site contains a hidden form or image tag that submits a request to:

```
https://yoursite.com/admin/moduleinterface.php?action=delete_holiday&hid=5
```

Because the admin's browser still has a valid CMSMS session cookie, the request is processed as if the admin performed it. Without CSRF protection, the holiday would be deleted.

### The {form\_start} Plugin

The simplest and recommended way to protect your forms is to use the `{form_start}` Smarty plugin. It automatically generates a `<form>` tag with all required hidden fields, including a CSRF token:

```smarty
{form_start}
  <!-- your form fields -->
  <input type="submit" name="{$actionid}submit" value="{$mod->Lang('submit')}" />
  <input type="submit" name="{$actionid}cancel" value="{$mod->Lang('cancel')}" />
{form_end}
```

The `{form_start}` plugin:

- Creates the `<form>` tag with the correct action URL.
- Sets the method to POST.
- Includes hidden fields for the module name, action ID, and return ID.
- Includes a CSRF token (`CMS_SECURE_PARAM_NAME`) that CMSMS validates on submission (admin only — not included on frontend forms).

The `{form_end}` plugin simply outputs the closing `</form>` tag.

### Passing Parameters Through {form\_start}

You can pass additional parameters that will be included as hidden form fields:

```smarty
{form_start hid=$holiday->id action='edit_holiday'}
```

These parameters are automatically prefixed with `{$actionid}` and will appear in the `$params` array when the form is submitted.

### Admin Actions

For admin forms, CMSMS validates the CSRF token automatically when the form is submitted. You do not need to write any additional PHP code to check the token — just use `{form_start}` in your templates.

### Delete Actions and GET Requests

Delete links that use GET requests (clicking an icon in a list) are more vulnerable to CSRF because they don't go through a form. Protect them with a JavaScript confirmation and consider using POST-based deletion instead:

```smarty
<!-- Approach 1: JavaScript confirmation (basic protection) -->
<script type="text/javascript">
$(document).ready(function(){
    $('a.del_item').click(function(){
        return confirm('{$mod->Lang('confirm_delete')}');
    });
});
</script>

<a class="del_item"
   href="{cms_action_url action=delete_holiday hid=$holiday->id}"
   title="{$mod->Lang('delete')}">{admin_icon icon='delete.gif'}</a>
```

For stronger protection, use a small POST form for each delete action instead of a GET link:

```smarty
<!-- Approach 2: POST form with CSRF token (stronger) -->
{form_start action='delete_holiday' hid=$holiday->id}
  <button type="submit" onclick="return confirm('{$mod->Lang('confirm_delete')}')"
          title="{$mod->Lang('delete')}">{admin_icon icon='delete.gif'}</button>
{form_end}
```

### Frontend Forms

**Important:** The `{form_start}` plugin does **not** include a CSRF token on frontend forms. It generates the correct action URL and hidden fields for module routing, but the `CMS_SECURE_PARAM_NAME` token is only added for admin requests.

This means frontend forms created with `{form_start}` are not CSRF-protected by default:

```smarty
{form_start action='submit_entry'}
  <!-- No CSRF token is included here -->
  <input type="text" name="{$actionid}name" value="" />
  <input type="submit" name="{$actionid}submit" value="Submit" />
{form_end}
```

To add CSRF protection to frontend forms, use the `{xt_form_csrf}` tag provided by the [CMSMSExt](https://forge.cmsmadesimple.org/projects/cmsmext) module:

```smarty
{form_start action='submit_entry'}{xt_form_csrf}
  <input type="text" name="{$actionid}name" value="" />
  <input type="submit" name="{$actionid}submit" value="Submit" />
{form_end}
```

Then validate the token in your action handler:

```php
// in action.submit_entry.php
if (!\xt_utils::valid_form_csrf()) {
    throw new \RuntimeException($this->Lang('error_security'));
}
```
> **Note:** The CMSMS core does not currently provide a built-in CSRF mechanism for frontend forms. Until one is available, CMSMSExt is the recommended solution.

### Summary

| Scenario | Protection method |
| --- | --- |
| Admin forms (add, edit, settings) | Use `{form_start}` / `{form_end}` — CSRF token is automatic |
| Admin delete via link | JavaScript confirmation; consider POST form instead |
| Frontend forms | Use `{form_start}` / `{form_end}` for routing, add `{xt_form_csrf}` (CMSMSExt) for CSRF protection |

### Next Steps

Continue to [Securing (Escaping) Output](/docs/security/securing-output) to learn how to prevent XSS vulnerabilities when displaying data.
