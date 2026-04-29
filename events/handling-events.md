## Handling Events

Your module can listen for events sent by other modules or by the CMSMS core. When the event fires, CMSMS calls your module's `DoEvent()` method with the event data.

### Step 1: Register as a Handler

Register your module as an event handler in `method.install.php`:

```php
// method.install.php
if (!defined('CMS_VERSION')) exit;

// Listen for core events
$this->AddEventHandler('Core', 'ContentPostRender');

// Listen for another module's events
$this->AddEventHandler('Holidays', 'HolidayAdded');
```

The first argument is the name of the module that sends the event (use `'Core'` for CMSMS core events). The second argument is the event name.

An optional third parameter controls whether the handler can be removed by an admin from the Events panel:

```php
// Non-removable handler (admin cannot detach it)
$this->AddEventHandler('Core', 'ContentPostRender', false);
```

### Step 2: Tell CMSMS Your Module Handles Events

Override the `HandlesEvents()` method in your module class to return `true`:

```
public function HandlesEvents() { return true; }
```

### Step 3: Implement DoEvent()

Override the `DoEvent()` method to handle incoming events. CMSMS calls this method whenever an event your module is registered for is fired:

```php
public function DoEvent($originator, $eventname, &$params)
{
    switch ($eventname) {
        case 'ContentPostRender':
            // Do something after content is rendered
            // $params['content'] contains the rendered content
            break;

        case 'HolidayAdded':
            // React to a new holiday being created
            $holiday_id = $params['holiday_id'];
            $name = $params['name'];
            // e.g., send a notification, update a cache, log it
            break;
    }
}
```

#### DoEvent() parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `$originator` | string | The name of the module that sent the event, or `'Core'` |
| `$eventname` | string | The name of the event |
| `&$params` | array | Associative array of event data. Passed by reference — some events allow you to modify the data |

> **Note:** The `$params` array is passed by reference. For some events (like `ContentPostRender`), you can modify the data and the changes will be reflected in the calling code. Check the event documentation to see which parameters are modifiable.

### Removing Event Handlers

Remove your handlers in `method.uninstall.php`:

```php
// method.uninstall.php
$this->RemoveEventHandler('Core', 'ContentPostRender');
$this->RemoveEventHandler('Holidays', 'HolidayAdded');
```

### UDTs as Event Handlers

Events can also be handled by User Defined Tags (UDTs). This is configured through the admin panel under Extensions > Events, not in module code. UDT handlers are useful for site-specific customizations that don't warrant a full module.

### Handler Execution Order

When an event fires, handlers are called in the order they were registered. The admin can reorder handlers from the Events panel. If your handler depends on another handler running first, document this requirement clearly.

### Soft Dependencies

Handling events from another module does not create a hard dependency. If the other module is not installed, your handler simply never gets called. This is the recommended pattern for optional integrations:

```php
// In method.install.php — this is safe even if Holidays is not installed
// The handler will only fire if Holidays is installed and sends the event
$this->AddEventHandler('Holidays', 'HolidayAdded');
```

Compare this with a hard dependency via `GetDependencies()`, which requires the other module to be installed before yours can be installed.

### Complete Example

```php
// Holidays.module.php
class Holidays extends CMSModule
{
    // ...

    public function HandlesEvents() { return true; }

    public function DoEvent($originator, $eventname, &$params)
    {
        if ($originator === 'Core' && $eventname === 'ContentPostRender') {
            // Inject holiday banner into page content
            $today = date('Y-m-d');
            $db = \cms_utils::get_db();
            $sql = 'SELECT name FROM ' . CMS_DB_PREFIX . 'mod_holidays
                    WHERE the_date = ? AND published = 1';
            $name = $db->GetOne($sql, [strtotime($today)]);
            if ($name) {
                $params['content'] = '&lt;div class="holiday-banner"&gt;'
                    . htmlspecialchars($name) . '&lt;/div&gt;'
                    . $params['content'];
            }
        }
    }
}

// method.install.php
$this->AddEventHandler('Core', 'ContentPostRender');

// method.uninstall.php
$this->RemoveEventHandler('Core', 'ContentPostRender');
```

### Next Steps

Continue to [Core Events Reference](/modules/events/core-events-reference/) for a list of events sent by the CMSMS core that your module can handle.
