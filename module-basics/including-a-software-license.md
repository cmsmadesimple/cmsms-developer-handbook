## Including a Software License

All CMSMS modules distributed through the Forge must be released under a GPL-compatible license. This is a requirement of the CMS Made Simple project itself, which is licensed under the GNU General Public License (GPL).

### Why Licensing Matters

A software license tells other developers and users what they can and cannot do with your code. Without a license, your code is technically "all rights reserved" — meaning nobody can legally use, modify, or distribute it, even if you share it publicly.

For CMSMS modules specifically:

- The CMSMS core is GPL-licensed. Modules that extend it are generally considered derivative works and must use a GPL-compatible license.
- The Forge requires a GPL-compatible license for all submitted modules.
- A clear license encourages adoption, contributions, and trust from the community.

### Recommended License

The most common choice for CMSMS modules is the **GNU General Public License v3** (GPLv3) or later. This is the same license family used by the CMSMS core.

Other GPL-compatible licenses that are acceptable include:

- GNU General Public License v2 (GPLv2)
- GNU Lesser General Public License (LGPL)
- MIT License
- BSD License (2-clause or 3-clause)

### How to Include Your License

#### 1. Add a LICENSE file

Place a `LICENSE` file in the `docs/` directory of your module:

```
modules/YourModule/
├── YourModule.module.php
├── docs/
│   ├── help.inc
│   ├── changelog.inc
│   └── LICENSE
└── ...
```

You can obtain the full GPLv3 text from <https://www.gnu.org/licenses/gpl-3.0.txt>.

#### 2. Add a header comment to your PHP files

Each PHP file should include a brief header identifying the module, author, copyright, and license:

```php
&lt;?php
#--------------------------------------------------
# Module: Holidays
# Author: Your Name
# Copyright: (C) 2025 Your Name, you@example.com
# Licence: GNU General Public License version 3
# See LICENSE for full license information.
#--------------------------------------------------
if (!defined('CMS_VERSION')) exit;
```

This header serves two purposes:

- It makes the license immediately visible to anyone reading the source code.
- It identifies the author and copyright holder, which is important if the module is forked or redistributed.

### What the License Means in Practice

Under the GPLv3:

- **Users can** install, use, modify, and redistribute your module freely.
- **Users must** keep the license and copyright notices intact when redistributing.
- **Users must** release any modifications under the same GPL license if they distribute the modified version.
- **You retain** copyright over your original code. The GPL does not transfer ownership.

### Forking and Redistribution

Because all CMSMS modules use GPL-compatible licenses, any developer can fork an existing module — rename it, modify it, and redistribute it — as long as they comply with the license terms. This is by design and is a strength of the CMSMS ecosystem.

If you fork another developer's module:

- Keep the original copyright and license notices.
- Add your own copyright notice for your modifications.
- Release under the same (or compatible) license.
- Rename the module to avoid confusion with the original.

### Next Steps

This completes the Module Basics chapter. You now have the foundation to build a well-structured, properly licensed CMSMS module. Continue to [Module Security](/modules/security/) to learn how to protect your module and its users.
