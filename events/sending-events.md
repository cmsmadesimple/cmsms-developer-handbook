## Sending Events

Your module can define its own events that other modules or UDTs can listen for. This allows other developers to extend your module's behavior without modifying your code.

### Step 1: Create the Event

Register your event during installation in `method.install.php`. This tells CMSMS that your module can send this event:

```php
// method.install.php
if (!defined('CMS_VERSION')) exit;

$this->CreateEvent('HolidayAdded');
$this->CreateEvent('HolidayEdited');
$this->CreateEvent('HolidayDeleted');
```

The `CreateEvent()` method on your module instance automatically associates the event with your module name. Behind the scenes, it calls `Events::CreateEvent('Holidays', 'HolidayAdded')`.

### Step 2: Send the Event

Fire the event at the appropriate point in your code, passing any relevant data as an associative array:

```php
// In action.edit_holiday.php, after saving
$holiday->save();

$this->SendEvent('HolidayAdded', ['holiday_id' => $holiday->id, 'name' => $holiday->name]);

$this->SetMessage($this->Lang('holiday_saved'));
$this->RedirectToAdminTab();
```

The `SendEvent()` method calls all registered handlers for this event, passing them the parameters array. Handlers are called synchronously — your code waits until all handlers have finished before continuing.

### Step 3: Remove Events on Uninstall

Clean up your events in `method.uninstall.php`:

```php
// method.uninstall.php
if (!defined('CMS_VERSION')) exit;

$this->RemoveEvent('HolidayAdded');
$this->RemoveEvent('HolidayEdited');
$this->RemoveEvent('HolidayDeleted');
```

The `RemoveEvent()` method removes the event and all its registered handlers from the database. Only events created by your module can be removed by your module.

### Providing Event Documentation

Override `GetEventDescription()` and `GetEventHelp()` in your module class to provide documentation for your events. These are displayed in the admin panel under Extensions > Events:

```php
public function GetEventDescription($eventname)
{
    return $this->Lang('event_desc_' . $eventname);
}

public function GetEventHelp($eventname)
{
    return $this->Lang('event_help_' . $eventname);
}
```

And in your `lang/en_US.php`:

```
$lang['event_desc_HolidayAdded'] = 'Sent after a new holiday is created';
$lang['event_help_HolidayAdded'] = 'Parameters: holiday_id (int) - The ID of the new holiday, name (string) - The holiday name';
```

### Adding Events in Upgrades

If you add new events in a later version, create them in `method.upgrade.php`:

```php
if (version_compare($oldversion, '1.2', '&lt;')) {
    $this->CreateEvent('HolidayPublished');
}
```

### Complete Example

```php
// method.install.php
$this->CreateEvent('HolidayAdded');

// action.edit_holiday.php
if (isset($params['submit'])) {
    $holiday->name = trim($params['name']);
    $holiday->save();
    $this->SendEvent('HolidayAdded', [
        'holiday_id' => $holiday->id,
        'name'       => $holiday->name,
        'the_date'   => $holiday->the_date,
    ]);
    $this->SetMessage($this->Lang('holiday_saved'));
    $this->RedirectToAdminTab();
}

// method.uninstall.php
$this->RemoveEvent('HolidayAdded');
```

### Next Steps

Continue to [Handling Events](/modules/events/handling-events/) to learn how to listen for and respond to events from other modules and the CMSMS core.
