## Module Check

Module Check is a diagnostic module that scans installed CMSMS modules for compliance with best practices, security patterns, deprecated API usage, and structural requirements — all from within the admin panel.

### Installation

1. Upload the `ModuleCheck` folder to the `modules/` directory.
2. Install via Extensions > Module Manager.
3. Grant the "Manage Module Check" permission to admin users.
4. Navigate to Extensions > Module Check.

### What It Checks

Module Check runs 19 automated checks across 3 categories:

#### General

- **Module Class** — Verifies required methods and structure of the main module class.
- **Module Structure** — Ensures required files and directories exist.
- **Module Info** — Validates moduleinfo.ini completeness and correctness.
- **Install / Uninstall** — Validates install/uninstall/upgrade methods exist and are correct.
- **Method Signatures** — Validates method signatures match CMSMS API expectations.
- **Module Compliance** — Error reporting, debug output, external services, metadata.

#### Security

- **File Headers** — Ensures files have proper headers and no BOM/closing tags.
- **Admin Permissions** — Verifies proper permission checks in admin actions.
- **Security** — Scans for SQL injection, XSS, and other security issues.
- **Security Advanced** — Input-to-sink flow analysis (SQL, XSS, file, shell, include), CSRF, credentials.
- **Templates** — Validates Smarty templates for best practices.
- **Code Obfuscation** — Detects obfuscated/encoded code (base64, eval, etc.).

#### Best Practices

- **File Types** — Checks for disallowed file types in the module.
- **Localhost** — Detects hardcoded localhost/127.0.0.1 references.
- **PRG Pattern** — Checks admin actions follow Post-Redirect-Get pattern.
- **Deprecated PHP** — Flags deprecated PHP functions and CMSMS API calls.
- **Naming Conventions** — Generic class names, preference keys, permissions, events, Smarty vars.
- **Code Quality** — DB queries in loops, error suppression, deep nesting, empty catch blocks.
- **JSON Rules** — 68 pattern-based rules from the CMSMS Scanner rule set.

### Scoring

Each module receives a score from 0–100:

- Starts at 100 points.
- Each unique error deducts `severity × 2` points.
- Each unique warning deducts `severity × 1` point.
- **Pass** — no errors, score ≥ 70.
- **Warning** — no errors, but has warnings.
- **Fail** — any errors, or score < 70.

### Usage

1. Navigate to Extensions > Module Check.
2. Select a module from the dropdown.
3. Optionally filter by category (General, Security, Best Practices) or type (error, warning).
4. Click "Run Checks" to scan the module.
5. Review the results table showing findings with severity, type, message, and file.

Module Check is read-only — it scans files but never modifies them.

### Next Steps

Continue to {cms\_selflink dir='next' text='CMSMS Scanner'} for a standalone scanner with deeper analysis capabilities.
