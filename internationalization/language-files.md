## Language Files

Language files store all human-readable strings used by your module. The base language file is `lang/en_US.php` — it serves as both the English translation and the master template for all other translations.

### File Location

```
modules/YourModule/
├── lang/
│   ├── en_US.php          # Base English strings (required)
│   └── ext/               # Community translations
│       ├── fr_FR.php      # French
│       ├── de_DE.php      # German
│       └── he_IL.php      # Hebrew
```

The `en_US.php` file goes directly in `lang/`. Translations from the Translation Center go in `lang/ext/`.

### File Format

```php
&lt;?php
// lang/en_US.php

$lang['friendlyname'] = 'Holidays';
$lang['admindescription'] = 'A module for managing and displaying holidays';
$lang['ask_uninstall'] = 'Are you sure you want to uninstall the Holidays module? All holiday data will be permanently deleted.';

// Form labels
$lang['name'] = 'Name';
$lang['date'] = 'Date';
$lang['description'] = 'Description';
$lang['published'] = 'Published';

// Actions
$lang['submit'] = 'Submit';
$lang['cancel'] = 'Cancel';
$lang['add_holiday'] = 'Create a New Holiday';
$lang['edit'] = 'Edit this';
$lang['delete'] = 'Delete this';

// Messages
$lang['holiday_saved'] = 'This holiday is now saved';
$lang['holiday_deleted'] = 'This holiday is now deleted';
$lang['confirm_delete'] = 'Are you sure that you want to delete this holiday record?';

// Errors
$lang['error_name_required'] = 'The holiday name is required';
$lang['error_invalid_date'] = 'Please enter a valid date';
$lang['sorry_noholidays'] = 'Sorry, we could not find any holidays that match the specified criteria';
$lang['error_notfound'] = 'The Holiday specified could not be displayed';

// Parameter help
$lang['param_hid'] = 'Applicable only to the detail action, this parameter accepts the integer id of a holiday to display';
$lang['param_limit'] = 'Applicable only to the default action, this parameter limits the number of holidays displayed';
$lang['param_detailpage'] = 'Applicable only to the default action, this parameter allows specifying an alternate page alias on which to display the detail results';
```

### Conventions

- Use lowercase keys with underscores: `holiday_saved`, not `HolidaySaved`.
- Group related strings together with comments.
- Keep the file open in your editor while developing — you'll add strings frequently.
- Do not include a closing `<?>` tag.
- The `friendlyname` and `admindescription` keys are expected by the CMSModule methods `GetFriendlyName()` and `GetAdminDescription()`.

### How CMSMS Loads Language Files

1. The first time `Lang()` is called, CMSMS reads `lang/en_US.php`.
2. If the current language (based on admin user preference or frontend locale) is not `en_US`, CMSMS also reads the corresponding file from `lang/ext/` (e.g., `lang/ext/fr_FR.php`).
3. Translation strings override the English defaults. Any keys not present in the translation fall back to English.

### Next Steps

Continue to [Using Lang Strings](/modules/internationalization/using-lang-strings/) to learn how to access translations in your code and templates.
