## Submitting Your Module

Before submitting your module to the Forge, ensure it meets the quality standards expected by the CMSMS community.

### Pre-Submission Checklist

- **License** — Include a GPL-compatible license file in `docs/LICENSE`. Add license headers to all PHP files.
- **Security** — Every PHP file starts with `if (!defined('CMS_VERSION')) exit;`. Every admin action checks permissions. All SQL uses parameterized queries.
- **Install / Uninstall** — The module installs cleanly and uninstalls completely, removing all tables, permissions, preferences, and events.
- **Language strings** — All user-facing text uses `Lang()`. The `lang/en_US.php` file is complete.
- **Help** — Provide module help via `GetHelp()` (or a `docs/help.inc` file) and a changelog via `GetChangeLog()`.
- **Parameter documentation** — All frontend parameters are registered with `SetParameterType()` and documented with `CreateParameter()`.
- **No hardcoded paths** — Use `CMS_DB_PREFIX`, `GetModulePath()`, `GetModuleURLPath()`, and other API methods.
- **No localhost references** — Remove any hardcoded `localhost`, `127.0.0.1`, or development URLs.
- **No debug output** — Remove `var_dump()`, `print_r()`, `error_log()`, and similar debug statements.
- **No closing PHP tags** — Omit `<?>` at the end of PHP files.
- **Test on a clean install** — Install, use, upgrade, and uninstall on a fresh CMSMS installation.

### Using Module Check

Run the [Module Check](/modules/developer-tools/module-check/) module against your module before submission. It performs automated checks for all of the above issues and more, giving you a score from 0–100 with specific findings.

### Registering on the Forge

1. Create an account on [dev.cmsmadesimple.org](https://dev.cmsmadesimple.org/).
2. Register a new project for your module.
3. Upload your module package (XML or ZIP).
4. Provide a description, screenshots, and documentation.

### Maintaining Your Module

- Respond to bug reports and feature requests on the Forge.
- Test your module against new CMSMS releases.
- Pull translations from the Translation Center before each release.
- Increment the version number for every release, even minor fixes.
- Maintain a changelog so users know what changed.

### Next Steps

Continue to [Module XML Packaging](/modules/cmsms-forge/module-xml-packaging/) to learn how CMSMS packages modules for distribution.
