## Content Block Tags

CMSMS page templates use content block tags like `{content}` to define editable regions. Modules can create custom content block types that appear as special fields in the page editor — for example, a date picker, a file selector, or a module-specific data chooser.

### How Content Blocks Work

In a page template, the `{content}` tag defines the main editable area. Additional content blocks can be created with the `{content_module}` tag:

```smarty
&lt;!-- Standard content block --&gt;
{content}

&lt;!-- Additional text block --&gt;
{content block=sidebar label='Sidebar Content'}

&lt;!-- Module-provided content block --&gt;
{content_module module=Holidays block=featured_holiday label='Featured Holiday'}
```

When a module is specified with `{content_module}`, CMSMS calls your module to provide the input field in the page editor and to render the value on the frontend.

### Implementing a Content Block Type

To provide a custom content block type, override these four methods in your module class:

#### GetContentBlockFieldInput()

Returns the HTML for the input field displayed in the page editor:

```php
public function GetContentBlockFieldInput($blockName, $value, $params, $adding, ContentBase $content_obj)
{
    // Return an HTML input for the page editor
    $holidays = $this->getPublishedHolidays();
    $out = '&lt;select name="' . $blockName . '"&gt;';
    $out .= '&lt;option value=""&gt;-- Select --&lt;/option&gt;';
    foreach ($holidays as $h) {
        $sel = ($value == $h->id) ? ' selected' : '';
        $out .= '&lt;option value="' . $h->id . '"' . $sel . '&gt;'
              . htmlspecialchars($h->name) . '&lt;/option&gt;';
    }
    $out .= '&lt;/select&gt;';
    return $out;
}
```

#### GetContentBlockFieldValue()

Extracts and returns the submitted value from the page editor form:

```php
public function GetContentBlockFieldValue($blockName, $blockParams, $inputParams, ContentBase $content_obj)
{
    if (isset($inputParams[$blockName])) {
        return (int) $inputParams[$blockName];
    }
    return false;
}
```

#### ValidateContentBlockFieldValue()

Validates the value before saving. Return an error message string to prevent saving, or empty string to allow it:

```php
public function ValidateContentBlockFieldValue($blockName, $value, $blockparams, ContentBase $content_obj)
{
    if (!empty($value) && !HolidayItem::load_by_id((int) $value)) {
        return $this->Lang('error_invalid_holiday');
    }
    return '';
}
```

#### RenderContentBlockField()

Renders the stored value on the frontend:

```php
public function RenderContentBlockField($blockName, $value, $blockparams, ContentBase $content_obj)
{
    if (empty($value)) return '';
    $holiday = HolidayItem::load_by_id((int) $value);
    if (!$holiday) return '';
    return '&lt;div class="featured-holiday"&gt;'
         . htmlspecialchars($holiday->name)
         . ' &amp;mdash; ' . date('F j, Y', $holiday->the_date)
         . '&lt;/div&gt;';
}
```

### Using the Content Block in a Template

Once implemented, template designers use `{content_module}` to place your custom block:

```smarty
&lt;div class="hero"&gt;
  {content_module module=Holidays block=featured_holiday label='Featured Holiday'}
&lt;/div&gt;
```

In the page editor, the admin will see a dropdown of holidays (from your `GetContentBlockFieldInput`). On the frontend, the selected holiday is rendered by your `RenderContentBlockField`.

### Method Summary

| Method | Called when | Returns |
| --- | --- | --- |
| `GetContentBlockFieldInput()` | Page editor displays the field | HTML for the input control |
| `GetContentBlockFieldValue()` | Page editor form is submitted | The cleaned value to store |
| `ValidateContentBlockFieldValue()` | Before saving the page | Error message string, or empty to allow |
| `RenderContentBlockField()` | Frontend page is rendered | HTML output for the frontend |

### Next Steps

Continue to [Tag Parameters](/tag-parameters) to learn how to define, register, and document the parameters your tags accept.
