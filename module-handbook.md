## Module Handbook

Welcome to the CMS Made Simple Module Handbook. This resource is the definitive guide for developers who want to build, maintain, and distribute modules for CMS Made Simple (CMSMS).

Modules are the primary way to extend the functionality of CMS Made Simple. They can add new content types, admin tools, frontend features, API integrations, and much more — all without modifying the core system.

### What You'll Learn

This handbook walks you through every aspect of module development, from creating your first module to publishing it on the CMSMS Forge:

- **Introduction to Module Development** — Understand what modules are, how they fit into the CMSMS architecture, and what you can build with them.
- **Module Basics** — Learn the required file structure, header requirements, install/uninstall/upgrade lifecycle methods, directory paths, and licensing.
- **Module Security** — Protect your module and its users with proper permission checks, input sanitization, output escaping, CSRF protection, and data validation.
- **Events** — Send and handle events (CMSMS's hook system) to let modules communicate with each other and with the core.
- **Admin Interface** — Build admin panels using actions, tabs, and navigation within the CMSMS admin area.
- **Smarty Tags** — Create frontend tags that content editors can use in pages and templates.
- **Module Preferences** — Store and retrieve module settings using the Preferences API and site-wide preferences.
- **Database Operations** — Work with the database using ADODB, create and manage tables, and handle schema migrations across versions.
- **Background Jobs** — Schedule one-time and recurring background tasks using the CmsJobManager async job system.
- **Templates** — Use Smarty templates for both admin and frontend output, following CMSMS conventions.
- **Assets and Resources** — Load CSS, JavaScript, images, and icons in admin and frontend contexts.
- **Users and Permissions** — Integrate with the CMSMS permission system, create custom permissions, and check user roles.
- **Internationalization** — Make your module translatable with language files and the Lang() method.
- **The CMSMS Forge** — Package, submit, and maintain your module on the official CMSMS module repository.
- **HTTP and REST APIs** — Make external HTTP requests, build REST integrations, and comply with external service disclosure requirements.
- **Developer Tools** — Use diagnostic tools like Module Check and CMSMS Scanner to audit your module for best practices, security, and compliance.
- **Module Guidelines** — Reference of all compliance, security, and code quality rules enforced by Module Check.
- **Tutorial: Building a Holidays Module** — A hands-on, step-by-step guide to building a complete working module from scratch.

### Prerequisites

Before diving in, you should have:

- A working CMS Made Simple 2.2.x installation (2.2.16 or later recommended).
- PHP 8.2 or later.
- Basic knowledge of PHP, HTML, and SQL.
- Familiarity with the [Smarty template engine](https://www.smarty.net/docs/en/).
- Access to the `modules/` directory on your CMSMS installation.

### Hands-On Tutorial

> **Note:** **New to CMSMS module development?** Start with the [Tutorial: Building a Holidays Module](/tutorial-start). It walks you through creating a complete module from scratch in 7 step-by-step chapters. Or grab the [Starter Pack](/starter-pack) to download and install the finished module immediately.

### How This Handbook Is Organized

Each chapter focuses on a single topic and builds on the previous ones. You can read it front-to-back as a reference, or follow the hands-on tutorial first and then use the chapters to go deeper on specific topics. Code examples use real CMSMS API calls and follow current best practices.

Throughout this handbook, you'll see references to the `CMSModule` base class — this is the class every module extends. Understanding its methods and lifecycle is the foundation of CMSMS module development.

### Getting Help

- [CMSMS Forums](https://forum.cmsmadesimple.org/) — Community support and discussion.
- [CMSMS Documentation](https://docs.cmsmadesimple.org/) — Official documentation wiki.
- [CMSMS Forge](https://forge.cmsmadesimple.org/) — Module repository and issue tracking.
- [CMSMS on GitHub](https://github.com/cmsmadesimple) — Source code and contributions.
