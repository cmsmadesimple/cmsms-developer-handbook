## Module Assets (Icons, Banners, and Screenshots)

The `assets/` directory in your module is where you store images used on your Forge project page — the icon, banner, and screenshots that represent your module to potential users. These files are not used at runtime; they are purely for the Forge listing.

> **Important:** The `assets/` directory is **not** included in your module's XML package and should **not** be submitted with your module. These files are uploaded separately through the Forge project management interface. They exist in your development directory for convenience and version control, but the Module Manager packaging process excludes them entirely.

```
modules/YourModule/
└── assets/
    ├── icon.svg
    ├── icon-128x128.png
    ├── icon-256x256.png
    ├── banner-772x250.png
    ├── banner-1544x500.png
    ├── screenshot-1.png
    ├── screenshot-2.png
    └── screenshot-3.png
```

All filenames must be **lowercase**. Images that don't match the expected filenames or dimensions will not display on the Forge.

---

### Module Icon

The module icon is the primary visual identity of your module. It appears in Forge search results, your project page, and the CMSMS Module Manager.

| Filename | Dimensions | Purpose |
| --- | --- | --- |
| `icon.svg` | Any (vector, square) | Preferred — scales to any size, tiny file |
| `icon-128x128.png` | 128 × 128 px | Standard resolution fallback |
| `icon-256x256.png` | 256 × 256 px | High-DPI (Retina) displays |

**Rules:**

- Icons must be **square**.
- If you provide an SVG, you must also include `icon-128x128.png` as a fallback for older browsers and social media previews.
- If no icon is provided, a generic placeholder will be shown.
- PNG or SVG only — no GIF, no JPG for icons.
- Use a transparent background for PNG icons.
- Maximum file size: **1 MB** (aim for under 50 KB).
- Design for legibility at small sizes — avoid fine detail that disappears at 32×32.

**Where the icon appears:**

| Context | Display size |
| --- | --- |
| Forge search results | 64 × 64 px |
| Forge project page | 128 × 128 px |
| Module Manager list | 48 × 48 px |
| Admin navigation menu | 32 × 32 px |

---

### Banner (Header Image)

The banner is a wide image displayed at the top of your Forge project page. It gives your module a professional, branded appearance.

| Filename | Dimensions | Purpose |
| --- | --- | --- |
| `banner-772x250.png` | 772 × 250 px | Standard resolution |
| `banner-1544x500.png` | 1544 × 500 px | High-DPI (Retina) displays |

**Rules:**

- The standard `banner-772x250` image is required if you want a banner. The retina version is optional but recommended.
- The retina image is only used as an enhancement — you cannot use it alone without the 772×250 image.
- PNG or JPG format.
- Maximum file size: **4 MB** (aim for under 200 KB).
- Design so key elements are centered — edges may be cropped on narrow viewports.
- Avoid placing text at the extreme left or right edges.
- If no banner is provided, the Forge displays a default background.

**Design tips:**

- Use your module name and a simple visual that communicates what it does.
- Keep it clean — the banner is a branding element, not a feature list.
- Test at both sizes to ensure nothing important is lost at standard resolution.

---

### Screenshots

Screenshots show your module in action and appear in a gallery on your Forge project page. They help users understand what your module does before installing it.

| Filename | Dimensions | Purpose |
| --- | --- | --- |
| `screenshot-1.png` | 1200 px wide recommended | First screenshot |
| `screenshot-2.png` | 1200 px wide recommended | Second screenshot |
| `screenshot-N.png` | 1200 px wide recommended | Additional screenshots |

**Rules:**

- Filenames must be numbered sequentially: `screenshot-1`, `screenshot-2`, etc.
- PNG or JPG format.
- Maximum file size per screenshot: **2 MB** (aim for under 200 KB each).
- Recommended width: **1200 px** (will be scaled down for display).
- Maximum recommended dimensions: **1920 × 1080 px**.
- There is no hard limit on the number of screenshots, but 3–6 is ideal.

**What to screenshot:**

1. The admin list view (showing data your module manages).
2. The admin edit/add form.
3. The frontend output (summary and detail views).
4. Any settings panel or configuration screen.
5. Notable features (drag-and-drop, AJAX interactions, etc.).

**Captions:**

Screenshot captions are pulled from your module's language file. Define them as:

```php
// lang/en_US.php
$lang['screenshot_1'] = 'Admin list view showing all holidays';
$lang['screenshot_2'] = 'Edit form with date picker and rich text editor';
$lang['screenshot_3'] = 'Frontend calendar display';
```

The Forge displays these captions below each screenshot in the gallery.

---

### Quick Reference

| Asset | Filename | Dimensions | Format | Max Size |
| --- | --- | --- | --- | --- |
| Icon (vector) | `icon.svg` | Square (any) | SVG | 1 MB |
| Icon (standard) | `icon-128x128.png` | 128 × 128 px | PNG | 1 MB |
| Icon (retina) | `icon-256x256.png` | 256 × 256 px | PNG | 1 MB |
| Banner (standard) | `banner-772x250.png` | 772 × 250 px | PNG / JPG | 4 MB |
| Banner (retina) | `banner-1544x500.png` | 1544 × 500 px | PNG / JPG | 4 MB |
| Screenshot | `screenshot-N.png` | 1200 px wide (rec.) | PNG / JPG | 2 MB |

---

### Next Steps

Once your assets are in place, see [Module XML Packaging](/docs/cmsms-forge/module-xml-packaging) to learn how these files are included in your distributable package.
