## Core Events Reference

The CMSMS core sends events at key points during content management, user administration, and page rendering. Your module can handle any of these events by registering a handler with `AddEventHandler('Core', 'EventName')`.

> **Note:** In CMSMS 2.2+, the core also uses the `HookManager` system. Hooks with names in the format `Module::EventName` automatically trigger the corresponding event. Both systems coexist — you can use either the Events API or the HookManager to listen for core events.

### Content Events

| Event Name | When It Fires | Key Parameters |
| --- | --- | --- |
| `ContentPreRender` | Before a content page is rendered | `content` (modifiable) |
| `ContentPostRender` | After a content page is rendered | `content` (modifiable) |
| `ContentEditPre` | Before a content page is saved in the admin | `content` (ContentBase object) |
| `ContentEditPost` | After a content page is saved in the admin | `content` (ContentBase object) |
| `ContentDeletePre` | Before a content page is deleted | `content` (ContentBase object) |
| `ContentDeletePost` | After a content page is deleted | `content` (ContentBase object) |
| `AddContentPre` | Before a new content page is created | `content` (ContentBase object) |
| `AddContentPost` | After a new content page is created | `content` (ContentBase object) |

### User and Group Events

| Event Name | When It Fires | Key Parameters |
| --- | --- | --- |
| `AddUserPre` | Before a new admin user is created | `user` (User object) |
| `AddUserPost` | After a new admin user is created | `user` (User object) |
| `EditUserPre` | Before an admin user is edited | `user` (User object) |
| `EditUserPost` | After an admin user is edited | `user` (User object) |
| `DeleteUserPre` | Before an admin user is deleted | `user` (User object) |
| `DeleteUserPost` | After an admin user is deleted | `user` (User object) |
| `AddGroupPre` | Before a new group is created | `group` (Group object) |
| `AddGroupPost` | After a new group is created | `group` (Group object) |
| `EditGroupPre` | Before a group is edited | `group` (Group object) |
| `EditGroupPost` | After a group is edited | `group` (Group object) |
| `DeleteGroupPre` | Before a group is deleted | `group` (Group object) |
| `DeleteGroupPost` | After a group is deleted | `group` (Group object) |
| `LoginAttempted` | When a login attempt is made, before credentials are verified *(coming soon)* | `user` (username string) |
| `LoginPre` | Before a successful admin login is finalized | `user` (User object) |
| `LoginPost` | After a successful admin login | `user` (User object) |
| `LoginVerified` | After login credentials have been verified and the session is established *(coming soon)* | `user` (User object) |
| `LoginFailed` | After a failed admin login attempt | `user` (username string) |
| `LogoutPre` | Before an admin logout | `user` (User object) |
| `LogoutPost` | After an admin logout | `user` (User object) |

### Design and Template Events

| Event Name | When It Fires | Key Parameters |
| --- | --- | --- |
| `AddTemplatePre` | Before a new template is created | `template` (CmsLayoutTemplate object) |
| `AddTemplatePost` | After a new template is created | `template` (CmsLayoutTemplate object) |
| `EditTemplatePre` | Before a template is edited | `template` (CmsLayoutTemplate object) |
| `EditTemplatePost` | After a template is edited | `template` (CmsLayoutTemplate object) |
| `DeleteTemplatePre` | Before a template is deleted | `template` (CmsLayoutTemplate object) |
| `DeleteTemplatePost` | After a template is deleted | `template` (CmsLayoutTemplate object) |
| `AddStylesheetPre` | Before a new stylesheet is created | `stylesheet` (CmsLayoutStylesheet object) |
| `AddStylesheetPost` | After a new stylesheet is created | `stylesheet` (CmsLayoutStylesheet object) |
| `EditStylesheetPre` | Before a stylesheet is edited | `stylesheet` (CmsLayoutStylesheet object) |
| `EditStylesheetPost` | After a stylesheet is edited | `stylesheet` (CmsLayoutStylesheet object) |

### Module Events

| Event Name | When It Fires | Key Parameters |
| --- | --- | --- |
| `ModuleInstalled` | After a module is installed | `name` (module name), `version` |
| `ModuleUninstalled` | After a module is uninstalled | `name` (module name) |
| `ModuleUpgraded` | After a module is upgraded | `name` (module name), `oldversion`, `newversion` |

### System Events

| Event Name | When It Fires | Key Parameters |
| --- | --- | --- |
| `SmartyPreCompile` | Before Smarty compiles a template | `content` (modifiable) |
| `SmartyPostCompile` | After Smarty compiles a template | `content` (modifiable) |

### Event Naming Pattern

Core events follow a consistent naming pattern:

- **Pre** events fire before the action occurs — you can inspect or modify the data.
- **Post** events fire after the action is complete — the data reflects the final state.

For example, `ContentEditPre` fires before a page is saved (you can modify the content object), while `ContentEditPost` fires after the save is complete (useful for logging, notifications, or cache invalidation).

### Viewing Events in the Admin Panel

You can see all registered events and their handlers in the CMSMS admin panel under Extensions > Events. This page shows:

- All core and module events.
- Which modules and UDTs are registered as handlers for each event.
- The ability to reorder handlers and attach UDTs to events.

### Next Steps

This completes the Events chapter. Continue to [Admin Interface](/modules/admin-interface/) to learn how to build admin panels with actions, tabs, and navigation.
