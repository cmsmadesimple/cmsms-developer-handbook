## cmscli - Command Line Interface

cmscli is a standalone PHP CLI application for managing CMSMS installations from the command line. It provides automation-friendly access to common administration tasks without requiring a web browser.

Modules can extend cmscli with their own commands using the `clicommands` capability.

### Installation

1. Download `cmscli.phar` from [dev.cmsmadesimple.org](http://dev.cmsmadesimple.org/projects/cmscli)
2. Upload it to your CMSMS root directory on the server (use binary/SFTP mode, not ASCII)
3. SSH into your server and run:

```bash
cd /path/to/your/cmsms
mv cmscli.phar cmscli
chmod +x cmscli
./cmscli --help
```

That's it. You run cmscli from inside your CMSMS root directory:

```bash
./cmscli site-info
./cmscli user-list
./cmscli module-list

# Or specify the CMSMS directory from elsewhere:
./cmscli --dir /path/to/cmsms site-info
```

### Built-in Commands

| Command | Description |
|---------|-------------|
| **Site** | |
| `site-info` | Display site name, CMSMS version, and schema version |
| `site-down` | Set the website to maintenance mode |
| `site-up` | Remove maintenance mode |
| `site-checksum-generate` | Generate a checksum file for integrity verification |
| `site-checksum-verify` | Verify installation integrity against a checksum file |
| `cache-clear` | Clear the server cache |
| **Configuration** | |
| `config-get` | Get a CMSMS config variable value |
| `config-set` | Set a CMSMS config variable in config.php |
| **Database** | |
| `db-dump` | Generate a mysqldump of the CMSMS database |
| `db-import` | Import an SQL file into the database |
| `db-reset` | Drop tables from the database |
| **Users and Groups** | |
| `user-list` | List admin users |
| `user-add` | Create a new admin user |
| `user-del` | Remove an admin user |
| `user-edit` | Edit admin account details |
| `user-password` | Change an admin user's password |
| `user-activate` | Activate a deactivated admin account |
| `user-deactivate` | Deactivate an admin account |
| `user-group` | Add or remove a user from admin groups |
| `group-list` | List admin groups |
| `group-add` | Create a new admin group |
| `group-del` | Remove an admin group |
| `group-perm-list` | List permissions for a group |
| **Modules** | |
| `module-list` | List installed modules |
| `module-install` | Install a module |
| `module-upgrade` | Upgrade a module |
| `module-activate` | Activate a module |
| `module-deactivate` | Deactivate a module |
| `module-hide` | Hide a module from CMSMS |
| `module-unhide` | Un-hide a module |
| `module-list-hidden` | List hidden modules |
| **Designs, Templates, and Stylesheets** | |
| `design-list` | List all designs |
| `design-export` | Export a design to XML |
| `design-import` | Import a design from an XML file |
| `template-list` | List all templates |
| `template-export` | Export a template to stdout or file |
| `template-import` | Import template content from file or stdin |
| `stylesheet-list` | List all stylesheets |
| `stylesheet-export` | Export a stylesheet to stdout or file |
| `stylesheet-import` | Import stylesheet content from file or stdin |
| **Mail** | |
| `mail-info` | Display mail configuration |
| `mail-test` | Send a test email |
| **Other** | |
| `about` | Display information about cmscli |
| `cmsms-latestversion` | Get the latest released CMSMS version number |
| `cmsms-download` | Download the latest CMSMS installation assistant |

Use `./cmscli <command> --help` for detailed usage of any command.

### Adding CLI Commands to Your Module

Any module can contribute commands to cmscli by declaring the `clicommands` capability and implementing `get_cli_commands()`.

#### Step 1: Declare the Capability

In your main module class, override `HasCapability()`:

```php
public function HasCapability($capability, $params = [])
{
    if ($capability == 'clicommands') return true;
    return parent::HasCapability($capability, $params);
}
```

#### Step 2: Return Command Instances

Implement `get_cli_commands()` to return an array of command objects:

```php
public function get_cli_commands(\CMSMS\CLI\App $app)
{
    return [
        new MyModule\Commands\ItemListCommand($app),
        new MyModule\Commands\ItemExportCommand($app),
    ];
}
```

#### Step 3: Create Command Classes

Each command extends `\CMSMS\CLI\GetOptExt\Command`. Place them in your module's `lib/Commands/` directory.

```php
<?php
namespace MyModule\Commands;

use CMSMS\CLI\App;
use CMSMS\CLI\GetOptExt\Command;
use CMSMS\CLI\GetOptExt\Option;
use CMSMS\CLI\GetOptExt\GetOpt;
use GetOpt\Operand;

class ItemListCommand extends Command
{
    public function __construct(App $app)
    {
        // parent::__construct($app, 'command-name', $requires_cmsms = true);
        parent::__construct($app, 'mymodule-item-list');

        // Add options (flags and arguments)
        $this->addOption(
            Option::Create('b', 'brief')
                ->setDescription('Output only item names')
        );
        $this->addOption(
            Option::Create('t', 'type', GetOpt::REQUIRED_ARGUMENT)
                ->setDescription('Filter by type')
        );

        // Add operands (positional arguments)
        $this->addOperand(
            Operand::create('pattern', Operand::OPTIONAL)
        );
    }

    public function getShortDescription()
    {
        return 'List items managed by MyModule';
    }

    public function getLongDescription()
    {
        return <<<EOT
Lists all items in MyModule. Use --brief for names only.
Use --type to filter by item type.
EOT;
    }

    public function handle()
    {
        // Retrieve option and operand values
        $brief = $this->getOption('brief')->getValue();
        $type = $this->getOption('type')->getValue();
        $pattern = $this->getOperand('pattern')->getValue();

        // Access CMSMS APIs
        $db = $this->getDb();

        // Query your data
        $sql = 'SELECT * FROM ' . \CMS_DB_PREFIX . 'mod_mymodule_items';
        $params = [];
        if ($type) {
            $sql .= ' WHERE type = ?';
            $params[] = $type;
        }
        $rows = $db->GetArray($sql, $params);

        if (empty($rows)) return;

        // Output results
        if ($brief) {
            foreach ($rows as $row) {
                echo $row['name'] . "\n";
            }
            return;
        }

        $fmt = "%-6s %-40s %-20s\n";
        echo sprintf($fmt, 'ID', 'Name', 'Type');
        echo sprintf($fmt, '--', '----', '----');
        foreach ($rows as $row) {
            echo sprintf($fmt, $row['id'], $row['name'], $row['type']);
        }
    }
}
```

### Command API Reference

#### Constructor

```php
parent::__construct(App $app, string $command_name, bool $requires_cmsms = true);
```

The third parameter controls whether this command needs a CMSMS installation. Set to `false` only for commands that work independently (rare for module commands).

#### Available Methods in handle()

| Method | Returns | Description |
|--------|---------|-------------|
| `$this->getApp()` | `App` | The cmscli application instance |
| `$this->getDb()` | `Connection` | The CMSMS database connection |
| `$this->getOption('name')` | `Option` | Get an option by name (call `->getValue()` for the value) |
| `$this->getOperand('name')` | `Operand` | Get an operand by name (call `->getValue()` for the value) |

#### Options and Operands

Options are named flags with optional values. Operands are positional arguments.

```php
// Flag (boolean option, no argument)
$this->addOption(Option::Create('v', 'verbose')->setDescription('Verbose output'));

// Option with required argument
$this->addOption(Option::Create('f', 'file', GetOpt::REQUIRED_ARGUMENT)
    ->setDescription('Output file path'));

// Option with optional argument and default
$this->addOption(Option::Create('l', 'limit', GetOpt::OPTIONAL_ARGUMENT)
    ->setDescription('Result limit')
    ->setDefaultValue('50'));

// Required operand
$this->addOperand(Operand::create('name', Operand::REQUIRED));

// Optional operand
$this->addOperand(Operand::create('filter', Operand::OPTIONAL));
```

#### Output and Error Handling

- Write output to stdout with `echo`.
- Throw `\RuntimeException` for errors (cmscli catches and displays them).
- Exit codes: 0 for success (automatic), non-zero for errors (from exception code).
- Design for automation: minimal output, machine-parseable where possible.

```php
public function handle()
{
    $name = $this->getOperand('name')->getValue();
    if (!$name) {
        throw new \RuntimeException("Name is required");
    }

    // Do work...

    echo "Done.\n";
}
```

### Naming Conventions

Prefix your command names with the module name or a short identifier to avoid collisions:

- `mymodule-item-list`
- `mymodule-item-export`
- `mymodule-sync`

### File Organization

```
modules/MyModule/
  lib/
    Commands/
      class.ItemListCommand.php
      class.ItemExportCommand.php
      class.SyncCommand.php
```

Use your module's namespace for the command classes and let CMSMS autoloading handle them.

### Testing

Run your commands from the CMSMS root:

```bash
./cmscli --nopermcheck mymodule-item-list
./cmscli --nopermcheck mymodule-item-list --brief
./cmscli --nopermcheck mymodule-item-list --type article
```

Use `--nopermcheck` during development to skip file ownership checks.
