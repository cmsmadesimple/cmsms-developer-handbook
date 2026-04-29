## Hooks (HookManager)

CMSMS 2.2 introduced the `HookManager` — a modern hook system that complements the older Events API. Hooks support closures, callable functions, priority levels, and inline registration without needing database entries.

### Events vs Hooks

| Feature | Events API | HookManager |
| --- | --- | --- |
| Available since | CMSMS 1.x | CMSMS 2.2 |
| Stored in database | Yes | No (registered in code) |
| Manageable from admin panel | Yes (Extensions > Events) | No |
| Supports UDT handlers | Yes | No |
| Supports closures | No | Yes |
| Handler priorities | Order only (admin reorderable) | HIGH / NORMAL / LOW |
| Registration | method.install.php (persistent) | InitializeFrontend / InitializeAdmin (per-request) |
| Bridges to Events | N/A | Yes — hooks named `Module::EventName` automatically trigger the corresponding Event |

Both systems coexist. Use Events when you need admin-panel management or UDT handler support. Use Hooks for lightweight, code-only integrations — especially with closures.

### Sending a Hook

Fire a hook from your module using the static `do_hook()` method. The hook name should follow the `ModuleName::HookName` convention:

```
// Fire a hook after a successful checkout
$data = [
    'order_id'    => $order->id,
    'customer'    => $order->customer_email,
    'amount'      => $order->total,
];
\CMSMS\HookManager::do_hook('CMSMSStripe::CheckoutSuccess', $data);
```

#### Hook dispatch methods

The HookManager provides three ways to dispatch hooks, each with different behavior:

| Method | Behavior |
| --- | --- |
| `do_hook()` | Calls all handlers in priority order. Each handler can modify the data — output of one handler becomes input to the next. Also triggers the corresponding Event if the hook name uses `Module::EventName` format. |
| `do_hook_first_result()` | Calls handlers in order and returns the first non-empty result. Does not trigger Events. |
| `do_hook_accumulate()` | Calls all handlers and collects their return values into an array. Also triggers the corresponding Event. |

### Listening for a Hook

Register a handler using `add_hook()`, typically in your module's `InitializeFrontend()` or `InitializeAdmin()` method:

```php
public function InitializeFrontend()
{
    \CMSMS\HookManager::add_hook('CMSMSStripe::CheckoutSuccess', function($data) {
        // React to a successful Stripe checkout
        $order_id = $data['order_id'];
        $email = $data['customer'];

        // e.g., send a confirmation email, update inventory, log the sale
        $mailer = \cms_utils::get_module('CMSMailer');
        if ($mailer) {
            $mailer->SetBody("Order #{$order_id} confirmed.");
            $mailer->AddAddress($email);
            $mailer->Send();
        }
    });
}
```

#### Using a class method instead of a closure

```php
public function InitializeFrontend()
{
    \CMSMS\HookManager::add_hook('CMSMSStripe::CheckoutSuccess', [$this, 'handleCheckout']);
}

public function handleCheckout($data)
{
    // Handle the checkout event
}
```

### Handler Priorities

The third parameter to `add_hook()` sets the handler priority:

```
use \CMSMS\HookManager;

// High priority — runs first
HookManager::add_hook('CMSMSStripe::CheckoutSuccess', $callback, HookManager::PRIORITY_HIGH);

// Normal priority (default)
HookManager::add_hook('CMSMSStripe::CheckoutSuccess', $callback, HookManager::PRIORITY_NORMAL);

// Low priority — runs last
HookManager::add_hook('CMSMSStripe::CheckoutSuccess', $callback, HookManager::PRIORITY_LOW);
```

### The Event Bridge

When a hook name uses the `Module::EventName` format, `do_hook()` and `do_hook_accumulate()` automatically call `Events::SendEvent()` as well. This means:

- Handlers registered via `HookManager::add_hook()` will be called.
- Handlers registered via the Events API (`AddEventHandler()` / `DoEvent()`) will also be called.
- UDT handlers attached through the admin panel will also be called.

This bridge allows you to fire a single hook and reach both modern hook handlers and legacy event handlers.

### Checking if a Hook is Active

```
// Check if any hook is currently being processed
if (\CMSMS\HookManager::in_hook()) { ... }

// Check if a specific hook is being processed
if (\CMSMS\HookManager::in_hook('CMSMSStripe::CheckoutSuccess')) { ... }
```

This is useful to prevent recursive hook calls.

### When to Use Hooks vs Events

- **Use Hooks** when you want lightweight, code-only integration with closures or callables, and don't need admin-panel management.
- **Use Events** when you want the event to be visible and manageable in the admin panel (Extensions > Events), or when UDTs need to handle the event.
- **Use both** by naming your hook with the `Module::EventName` format — the HookManager will bridge to the Events system automatically.

### Complete Example

```php
// === Module A: CMSMSStripe (sends the hook) ===

// In action.process_payment.php
$data = ['order_id' => $order->id, 'amount' => $order->total];
\CMSMS\HookManager::do_hook('CMSMSStripe::CheckoutSuccess', $data);


// === Module B: OrderNotifier (listens via HookManager) ===

// In OrderNotifier.module.php
public function InitializeFrontend()
{
    \CMSMS\HookManager::add_hook('CMSMSStripe::CheckoutSuccess', function($data) {
        // Send notification email
    });
}


// === Module C: OrderLogger (listens via Events API) ===

// In method.install.php
$this->AddEventHandler('CMSMSStripe', 'CheckoutSuccess');

// In OrderLogger.module.php
public function HandlesEvents() { return true; }

public function DoEvent($originator, $eventname, &$params)
{
    if ($eventname === 'CheckoutSuccess') {
        // Log the order
    }
}
```

In this example, when CMSMSStripe fires the hook, both Module B (via HookManager) and Module C (via Events) receive the notification.

### Next Steps

This completes the Events chapter. Continue to [Admin Interface](/modules/admin-interface/) to learn how to build admin panels with actions, tabs, and navigation.
