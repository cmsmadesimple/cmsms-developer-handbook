## CMSMS Scanner

CMSMS Scanner is a standalone Python-based security scanner and audit tool for CMS Made Simple modules. It performs deep analysis of PHP and TPL files against a comprehensive rule set covering security vulnerabilities, deprecated API usage, naming conventions, code quality, and CMSMS compliance standards.

### Features

- **68 pattern-based rules** organized into CMSMS compliance, security, and code quality categories.
- **Heuristic analysis** for complex checks that require cross-line or module-level analysis.
- **Multiple input formats** — scan a module directory, an XML package, or a ZIP archive.
- **AI-powered reports** — optional integration with AWS Bedrock for automated audit reports and false positive detection.
- **JSON and human-readable output** for integration with CI/CD pipelines.

### Requirements

- Python 3.8+
- Access to the module files (directory, XML, or ZIP)
- Optional: AWS credentials for Bedrock agent integration

### Rule Categories

#### CMSMS Compliance (CMSMS\_CMS\_001–036)

Rules that check for CMSMS-specific patterns: proper file headers, CMS\_VERSION checks, permission checks in admin actions, correct method signatures, module structure, install/uninstall completeness, and more.

#### Security (CMSMS\_SEC\_001–018)

Rules that detect security vulnerabilities: SQL injection, XSS, CSRF, file inclusion, shell injection, hardcoded credentials, eval/exec usage, and unsafe file operations.

#### Code Quality (CMSMS\_QUAL\_001–014)

Rules that flag code quality issues: database queries in loops, error suppression operators, deep nesting, empty catch blocks, hardcoded localhost references, deprecated PHP functions, and naming convention violations.

### Rule Format

Each rule is a standalone JSON file with:

- `rule_id` — unique identifier (e.g., CMSMS\_SEC\_006).
- `detection.type` — "pattern" (regex) or "heuristic" (Python handler).
- `detection.patterns` — regex patterns for pattern-type rules.
- `exclude_if` — conditions that suppress false positives.
- `applies_to` — file types: "php", "tpl", or "module" (directory-level).
- `enabled` — boolean to toggle rules on/off.

### Shared Rules with Module Check

The CMSMS Scanner and Module Check share the same JSON rule set. Module Check includes these rules in its `rules/` directory and runs them via the `Check_JsonRules` class. This ensures consistent findings between the standalone scanner and the admin module.

### Usage

```
# Scan a module directory
python app.py /path/to/modules/Holidays

# Scan an XML package
python app.py /path/to/Holidays-1.0.xml

# Scan a ZIP archive
python app.py /path/to/Holidays-1.0.zip
```

### Integration with Development Workflow

- Run the scanner before every release to catch regressions.
- Integrate into CI/CD pipelines using JSON output.
- Use the AI-powered review agent to verify findings and reduce false positives.
- Track false positives in `reports/feedback.json` to improve accuracy over time.

For more information, see the CMSMS Scanner project documentation.
