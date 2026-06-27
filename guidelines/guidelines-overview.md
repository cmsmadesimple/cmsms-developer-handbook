## Module Guidelines

These guidelines define the standards every CMSMS module should meet before distribution. They are based on the 68 automated rules enforced by [ModuleChecker](/docs/developer-tools/module-checker).

Each guideline includes a rule ID, severity level, explanation of why it matters, how to fix it, and code examples showing the wrong and right way.

### In This Chapter

- [CMSMS Compliance Guidelines](/docs/guidelines/cmsms-compliance) — 34 rules covering naming conventions, API usage, module structure, deprecated methods, and CMSMS-specific patterns. (CMSMS\_CMS\_001–036)
- [Security Guidelines](/docs/guidelines/security-rules) — 18 rules covering SQL injection, XSS, file inclusion, command injection, CSRF, credentials, and code obfuscation. (CMSMS\_SEC\_001–018)
- [Code Quality Guidelines](/docs/guidelines/quality-rules) — 14 rules covering performance, separation of concerns, error handling, deprecated PHP functions, and file hygiene. (CMSMS\_QUAL\_001–014)

### Severity Levels

| Severity | Meaning |
| --- | --- |
| **high** | Must fix before distribution. Security vulnerabilities, broken installs, or major compliance failures. |
| **medium** | Should fix. Deprecated APIs, naming collisions, missing best practices. |
| **low** | Recommended improvement. Code style, minor optimizations, informational. |
| **info** | Informational only. Documentation and licensing suggestions. |
