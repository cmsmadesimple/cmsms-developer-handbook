## Module XML Packaging

CMSMS distributes modules as XML packages. When you install a module through Extensions > Module Manager, CMSMS downloads an XML file that contains all of the module's files encoded in a single package. Your module class includes a built-in method to generate this package.

### Generating an XML Package

The `CreateXMLPackage()` method on your module class generates the XML package string:

```php
$message = '';
$filecount = 0;
$xml = $this->CreateXMLPackage($message, $filecount);

if ($xml) {
    // $xml contains the full XML package string
    // $filecount contains the number of files included
    // $message may contain warnings
}
```

This method walks your module directory, encodes each file (base64 for binary files, CDATA for text), and wraps everything in an XML structure that includes your module metadata.

### What Gets Packaged

The XML package includes all files in your module directory, with some exceptions:

- Files and directories starting with `.` (e.g., `.git`, `.gitignore`) are excluded.
- The `lib/`, `lang/`, `templates/`, `docs/`, and other standard directories are included.
- Binary files (images, etc.) are base64-encoded.

> **Note:** Keep your module directory clean before packaging. Remove any development files, test fixtures, IDE configuration files, or build artifacts that should not be distributed.

### Package Structure

The XML package contains:

- Module metadata (name, version, author, dependencies, minimum CMSMS version).
- All module files with their relative paths and contents.
- File sizes and checksums for integrity verification.

### Distribution Methods

| Method | Description |
| --- | --- |
| CMSMS Forge | Upload the XML package to the Forge. Users install directly from Module Manager. |
| Manual XML install | Provide the XML file for download. Users upload it via Module Manager > "Install from file". |
| ZIP archive | Distribute as a ZIP of the module directory. Users extract it into `modules/` and install from Module Manager. |

### Next Steps

Continue to [Module Info File (moduleinfo.ini)](/modules/cmsms-forge/module-info-file/) to learn about the metadata file that describes your module.
