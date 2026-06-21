## ModuleChecker

ModuleChecker is a diagnostic module that scans installed CMSMS modules for compliance with best practices, security patterns, deprecated API usage, and structural requirements — all from within the admin panel.

**Forge Project:** [https://dev.cmsmadesimple.org/projects/modulechecker](https://dev.cmsmadesimple.org/projects/modulechecker)

### Installation

1. Upload the `ModuleChecker` folder to the `modules/` directory.
2. Install via Site Admin > Module Manager.
3. Grant the "Manage ModuleChecker" permission to admin users.
4. Navigate to Extensions > ModuleChecker.

The module automatically fetches the latest detection rules from GitHub on install.

### What It Checks

ModuleChecker runs automated checks across 3 categories, plus a JSON pattern rule engine that can be updated independently without a module upgrade.

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
- **Naming Conventions** — Generic class names, preference keys, permissions, events, Smarty vars.

#### Best Practices

- **File Types** — Checks for disallowed file types in the module.
- **Localhost** — Detects hardcoded localhost/127.0.0.1 references.
- **PRG Pattern** — Checks admin actions follow Post-Redirect-Get pattern.
- **Deprecated PHP** — Flags deprecated PHP functions and CMSMS API calls.
- **Code Quality** — DB queries in loops, error suppression, deep nesting, empty catch blocks.
- **JSON Rules** — Pattern-based rules from the CMSMS Scanner rule set, covering security, CMSMS compliance, and code quality patterns.

### Scoring

Each module receives a score from 0–100:

- Starts at 100 points.
- Each unique error deducts `severity × 2` points.
- Each unique warning deducts `severity × 1` point.
- **Pass** — no errors, score ≥ 70.
- **Warning** — no errors, but has warnings.
- **Fail** — any errors, or score < 70.

### Usage

1. Navigate to Extensions > ModuleChecker.
2. Select a module from the dropdown.
3. Optionally filter by category (General, Security, Best Practices) or type (error, warning).
4. Click "Run Checks" to scan the module.
5. Review the results table showing findings with severity, type, message, and file.

ModuleChecker is read-only — it scans files but never modifies them.

### Live Rules Updates

Detection rules are maintained in a separate public GitHub repository. The module checks for new rule versions on page load via the GitHub Tags API. If an update is available, a yellow notification banner appears in the Scanner tab with an "Update Now" button.

Clicking "Update Now" downloads the latest tagged release and applies it immediately. No module upgrade is needed to receive new rules or false positive fixes.

### .distignore Support

Modules can include a `.distignore` file in their root directory to exclude files and directories from scanning. This is useful for excluding distribution artifacts, VCS directories, and development files.

The format is one pattern per line. Lines starting with `#` are comments. Patterns can be exact filenames, directory names, or wildcard patterns (e.g. `*.zip`). All checks respect these exclusions.

### AI-Powered Reports

ModuleChecker can optionally generate detailed audit reports using the OpenAI API. After running a scan, click "Generate Report" to send the findings to OpenAI for analysis. The AI produces a structured markdown report with explanations, recommendations, and prioritized fixes.

Reports can be saved to the database and emailed directly to the scanned module's author.

This feature requires an OpenAI API key configured in the Settings tab. No data is sent without a configured key.

### Refactoring Plans

In addition to audit reports, ModuleChecker can generate refactoring plans that break down the work needed to resolve findings into actionable steps. Plans are saved and can be reviewed later from the history.

### Report History

Saved reports are stored in the database and accessible from the Reports tab. Each report records the module name, score, verdict, error/warning counts, and full scan data. Reports can be loaded, compared, and emailed.

### Settings

The Settings tab allows configuration of:

- **OpenAI API Key** — Required for AI report and refactoring plan generation. Validated on save.

### Third-Party Services

This module optionally connects to the following external services:

- **GitHub API** — Checks for and downloads detection rule updates. No user data is sent. Used automatically on page load (version check only) and when clicking "Update Now".
- **OpenAI API** — Generates AI-powered audit reports and refactoring plans. Scan findings (rule violations, file names, code snippets) are sent for analysis. Only used when explicitly triggered by the admin.

### Next Steps

Continue to {cms\_selflink dir='next' text='CMSMS Scanner'} for a standalone scanner with deeper analysis capabilities.
