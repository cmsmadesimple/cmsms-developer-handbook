## What is a Module?

A module is a self-contained package of PHP code that extends CMS Made Simple. Modules can create admin interfaces, display content on the frontend, manage database tables, handle events, define permissions, register Smarty tags, and much more.

Every module is a PHP class that extends the `CMSModule` base class. It lives in its own directory under the `modules/` folder of your CMSMS installation.

### Three Ways to Extend CMSMS

CMS Made Simple provides three mechanisms for adding PHP code to the system: Plugins, User Defined Tags (UDTs), and Modules. Understanding the differences helps you choose the right approach.

#### Plugins

Plugins are single PHP files placed in the `plugins/` directory. They are true Smarty plugins — function plugins, block plugins, modifiers, or compiler plugins. Plugins typically perform a single task like displaying or reformatting data, and should never write data anywhere.

Examples include the `{title}` and `{description}` plugins shipped with CMSMS that output fields of the current content page.

#### User Defined Tags (UDTs)

UDTs are Smarty plugins stored in the database. They are created and managed through the CMSMS admin console. Like plugins, they are normally small and perform a single task. UDTs are implemented as Smarty function plugins and can also be used as event handlers.

UDTs should not store data or output complex HTML — that is the job of templates.

#### Modules

Modules are larger sets of code that solve a complete problem. They can create and manage database tables, provide multiple views including forms, process form submissions, and offer extensive functionality. Modules can also do things that plugins and UDTs cannot.

### Capability Comparison

| Capability | Plugin | UDT | Module |
| --- | --- | --- | --- |
| A single PHP function | Yes | Yes | — |
| Can have an admin interface | No | No | Yes |
| Can handle events | No | Yes | Yes |
| Has install and uninstall | No | No | Yes |
| Handles routing (pretty URLs) | No | No | Yes |
| Can be shared on the Forge | No | No | Yes |
| Can create events | No | No | Yes |
| Can create page types | No | No | Yes |
| Can create PseudoCron tasks | No | No | Yes |
| Can be modifier / compiler / block | Yes | No | Yes* |
| Provide multiple plugins | No | No | Yes |
| Provide functions and classes for other modules | No | No | Yes |

\* Modules can register multiple different plugins of different types, though usually it is just a function plugin.

As the table shows, modules are the most powerful extensibility mechanism in CMSMS. They are usually the best choice when writing new functionality.

### Anatomy of a Module

At its simplest, a module is a directory containing a single PHP file with a class that extends `CMSModule`:

```
modules/
└── MyModule/
    └── MyModule.module.php
```

The PHP file contains:

```php
&lt;?php
if (!defined('CMS_VERSION')) exit;

class MyModule extends CMSModule
{
    public function GetVersion() { return '1.0'; }
    public function GetFriendlyName() { return 'My Module'; }
    public function GetAdminDescription() { return 'A sample module.'; }
    public function IsPluginModule() { return false; }
    public function HasAdmin() { return true; }
    public function GetAuthor() { return 'Your Name'; }
    public function GetAuthorEmail() { return 'you@example.com'; }
}
```

This is already a valid, installable module. It has no functionality yet, but CMSMS can detect it, install it, and display it in the admin navigation.

> **Note:** **Important naming rules:** The directory name, the file name, and the class name must all match exactly (e.g., `MyModule`). CMSMS is case-sensitive — always match the case consistently. The main module class itself cannot be in a namespace, though helper classes can be.

### What Modules Can Do

A fully developed module can:

- **Provide an admin panel** — with forms, lists, tabs, and navigation within the CMSMS admin console.
- **Display frontend content** — called from page templates using Smarty tags like `{cms_module module=MyModule}` or the shorthand `{MyModule}`.
- **Create database tables** — using the ADODB DataDictionary for database-agnostic schema management.
- **Define permissions** — to control which admin users can access module features.
- **Send and handle events** — to communicate with other modules and the CMSMS core.
- **Register routes** — for SEO-friendly (pretty) URLs on the frontend.
- **Integrate with Search** — by indexing content for the built-in Search module.
- **Store preferences** — both module-level and site-wide settings.
- **Support translations** — through language files and the CMSMS Translation Center.
- **Lazy load** — to avoid consuming memory on requests where the module isn't needed.
- **Create content types and block types** — extending the CMSMS content system itself.

### A Typical Module Structure

As a module grows, its directory structure expands to include actions, templates, language files, library classes, and lifecycle methods:

```
modules/MyModule/
├── MyModule.module.php        # Main module class (extends CMSModule)
├── action.defaultadmin.php    # Default admin action (admin panel entry point)
├── action.default.php         # Default frontend action
├── action.detail.php          # Additional frontend action
├── action.edit_item.php       # Additional admin action
├── action.delete_item.php     # Additional admin action
├── method.install.php         # Runs on module installation
├── method.uninstall.php       # Runs on module uninstallation
├── method.upgrade.php         # Runs when module version changes
├── moduleinfo.ini             # Module metadata for the Forge
├── lang/
│   └── en_US.php              # English language strings
│   └── ext/
│       └── fr_FR.php          # French translation (from Translation Center)
├── lib/
│   ├── class.MyItem.php       # Model class
│   └── class.MyQuery.php      # Query/repository class
└── templates/
    ├── defaultadmin.tpl       # Admin panel Smarty template
    ├── default.tpl            # Frontend summary template
    ├── detail.tpl             # Frontend detail template
    └── edit_item.tpl          # Admin edit form template
```

### The Module Lifecycle

Every module goes through a defined lifecycle managed by CMSMS:

1. **Detection** — CMSMS scans the `modules/` directory and finds your module class file.
2. **Installation** — An admin installs the module via Extensions > Module Manager. CMSMS executes `method.install.php`, where you create permissions, database tables, and default settings.
3. **Initialization** — On each request, CMSMS calls `InitializeFrontend()` or `InitializeAdmin()` depending on the context. This is where you register parameters, routes, and Smarty plugins.
4. **Action execution** — When a user navigates to your module (admin or frontend), CMSMS loads the appropriate `action.*.php` file and executes it.
5. **Upgrade** — When you change the version number returned by `GetVersion()`, CMSMS detects the mismatch and executes `method.upgrade.php`.
6. **Uninstallation** — An admin uninstalls the module. CMSMS executes `method.uninstall.php`, where you remove all tables, permissions, preferences, and other data your module created.

### Key Concepts

**Actions**
: Controllers in the MVC pattern. Each action is a separate PHP file (e.g.,action.defaultadmin.php) that handles a specific request. The special actiondefaultadminis the entry point for admin requests;defaultis the entry point for frontend requests.

**Templates**
: Views in the MVC pattern. Smarty.tplfiles in thetemplates/directory that render HTML. Actions load templates, assign data to them, and calldisplay().

**Models**
: CMSMS does not enforce a base model class. You create your own PHP classes in thelib/directory to represent your data. CMSMS provides an autoloader that automatically loads classes fromlib/.

**Parameters**
: Input values passed to actions. For admin requests, all parameters are available. For frontend requests, parameters must be registered viaSetParameterType()inInitializeFrontend()for security.

**Lang strings**
: Human-readable text stored inlang/en_US.phpand accessed via$this->Lang('key'). This enables translation to other languages.

### Next Steps

Now that you understand what modules are and what they can do, continue to [Module Basics](/modules/module-basics/) to learn the required file structure, header methods, and lifecycle hooks in detail.
