## Events

Events are CMSMS's hook system — they allow modules to communicate with each other and with the core without creating tight dependencies. A module can send an event when something happens, and any other module (or User Defined Tag) can listen for that event and respond.

This is the same pattern known as "hooks" or "signals" in other CMS platforms.

### In This Chapter

- [Sending Events](/modules/events/sending-events/) — Create and fire events from your module so other modules can react.
- [Handling Events](/modules/events/handling-events/) — Listen for events from other modules or the CMSMS core and respond to them.
- [Core Events Reference](/modules/events/core-events-reference/) — A reference of events sent by the CMSMS core that your module can handle.
- [Hooks (HookManager)](/modules/events/hooks/) — The modern hook system introduced in CMSMS 2.2, supporting closures, priorities, and inline registration.

### How Events Work

1. A module **creates** an event during installation — this registers the event name in the system.
2. Another module **registers a handler** for that event — either during installation or in its initialization method.
3. At runtime, the first module **sends** the event, optionally passing data as an associative array.
4. CMSMS calls all registered handlers in order, passing them the event data.

Events are one-to-many: one sender, any number of handlers. The sender does not need to know (or care) who is listening.

### Two Ways to Work with Events

CMSMS provides two interfaces for working with events:

- **CMSModule convenience methods** — `CreateEvent()`, `SendEvent()`, `AddEventHandler()`, `RemoveEvent()`, `RemoveEventHandler()`. These are called on your module instance (`$this->CreateEvent(...)`) and automatically pass your module name.
- **The Events class** — static methods like `Events::CreateEvent()`, `Events::SendEvent()`, `Events::ListEvents()`. These require you to pass the module name explicitly and provide additional methods like `ListEvents()` and `ListEventHandlers()`.

For most module development, the CMSModule convenience methods are all you need.
