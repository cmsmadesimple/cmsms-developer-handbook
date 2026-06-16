## Submitting Your Module

Before submitting your module to the Forge, ensure it meets the quality standards expected by the CMSMS community.

### Pre-Submission Checklist

- **Naming** — Follow the [Naming Conventions](/docs/module-basics/naming-conventions): PascalCase module name matching folder and class, prefixed database tables, prefixed permissions and events, and `MAJOR.MINOR.PATCH` version format.
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

<!--
### Using Module Check

Run the [Module Check](/docs/developer-tools/module-check) module against your module before submission. It performs automated checks for all of the above issues and more, giving you a score from 0–100 with specific findings.
-->

### Registering and Publishing on the Forge

This walkthrough covers the complete process, from building the XML package to uploading your first release.

#### Stage 1: Prepare the XML Package

Before touching the Forge, have your module's XML package built and version-stamped.

1. In the module's admin area, update the version number to the release you intend to publish (for example, `1.0`).
2. Click the yellow **XML** button on the Module Manager screen to generate and download the XML package.
3. Save the XML file somewhere accessible. You will upload it in Stage 5.

> You can also build a `.zip` of the module. The Module Manager only consumes the XML; the zip is purely for users who want to download the extracted files directly. XML only is fine.

#### Stage 2: Register the Project

1. Go to the [Forge](https://dev.cmsmadesimple.org) and open **My Page** (make sure you are logged in).
2. Click **Register a New Project**.
3. Fill in the **Name**. Use exactly the same name as in the Module Manager (e.g. `CMSSiteManager`). The Unix name auto-generates from this.

> The name cannot be changed after submission. Every other field can be edited later, so get the name right first time.

4. Leave **Project Purpose** short (a one-liner is fine). This field is not displayed publicly; it is mainly used in the notification email to moderators.
5. Write a thorough **Description**. This is the field that matters: it appears on the project page and is indexed into the Forge search database. Cover three things:
   - What the module does
   - How it works
   - Who should use it
6. Set **Project Type** to `Module`.
7. Set **License**. Check your module's `LICENSE` file. For GPL 3.0, select *GNU General Public License* from the top of the list.
8. Set **Repository Site** to GitHub (or wherever the code lives) and paste the repo URL in the format the form expects.
9. Click **Register**. If you get a validation error, it is almost always because Purpose was left blank.
10. Wait for a moderator to approve the project. You cannot add releases until approval clears.

#### Stage 3: Open the Admin Dashboard

1. Once approved, go back to **My Page**.
2. Click into the new project, then click **Admin Dashboard**.

From here you can re-edit every project field except the name: description, tags, repo URL, license, etc.

#### Stage 4: Create the File Package

You only do this once per project per channel.

1. On the Admin Dashboard, find **Add File Package**.
2. Create a package called `Stable Releases` (use that exact name). If you want to ship pre-release versions later, create a separate `Betas` package the same way.

> You can create the package now even if no files exist yet. Releases live inside it and are added in the next stage.

#### Stage 5: Add the First Release

1. Scroll down to the **Stable Releases** package, click **Edit Releases**, then **Add a Release**.
2. Set **Release Name** to the full semantic version, e.g. `1.0.0` (not just `1.0`).
3. Tick the bug tracker / change-tracking checkbox if you want issue tracking active for this release.
4. Set the **PHP version** compatibility range and the **CMSMS version** the release has been tested up to.
5. Click **Create New Release**.
6. On the release page that opens, upload the XML file from Stage 1. You can also upload a zip alongside it if you want to offer a pre-extracted download.

#### Stage 6: After Publishing

Once the XML is uploaded the module is live. Things worth knowing:

- The Module Manager pulls new releases from the XML. Users do not need the zip.
- Description edits propagate to the search database, so keep it accurate. It shapes how your module is discovered.
- If the module fails to appear in the Module Manager listing, report it on the Forge or in the developer channel.

#### Quick Checklist

- [ ] XML package built and version bumped
- [ ] Forge project registered with matching name and full description
- [ ] License, project type, repo URL filled in
- [ ] Approval received from moderator
- [ ] Stable Releases package created
- [ ] Release added with version number, tracker, PHP/CMSMS compatibility
- [ ] XML uploaded to the release

### Maintaining Your Module

- Respond to bug reports and feature requests on the Forge.
- Test your module against new CMSMS releases.
- Pull translations from the Translation Center before each release.
- Increment the version number for every release, even minor fixes.
- Maintain a changelog so users know what changed.

### Next Steps

Continue to [Module XML Packaging](/docs/cmsms-forge/module-xml-packaging) to learn how CMSMS packages modules for distribution.
