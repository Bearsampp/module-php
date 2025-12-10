<p align="center"><a href="https://bearsampp.com/contribute" target="_blank"><img width="250" src="img/Bearsampp-logo.svg"></a></p>

[![GitHub release](https://img.shields.io/github/release/bearsampp/module-php.svg?style=flat-square)](https://github.com/bearsampp/module-php/releases/latest)
![Total downloads](https://img.shields.io/github/downloads/bearsampp/module-php/total.svg?style=flat-square)

This is a module of [Bearsampp project](https://github.com/bearsampp/bearsampp) involving PHP.

## Build System

This project uses **Gradle** as its build system. The legacy Ant build has been fully replaced with a modern, pure Gradle implementation.

### Quick Start

```bash
# Display build information
gradle info

# List all available tasks
gradle tasks

# Verify build environment
gradle verify

# Build a release (interactive) - automatically fetches latest dependencies
gradle release

# Build a specific version (non-interactive)
gradle release -PbundleVersion=8.3.15

# Update dependencies only (without building)
gradle fetch -PbundleVersion=8.3.15

# Clean build artifacts
gradle clean
```

### Creating a New PHP Version Build

When a new PHP version is released, follow these steps:

1. **Create version directory structure**:
   ```bash
   mkdir bin/php8.4.16
   ```

2. **Copy configuration files** from a similar version:
   ```bash
   # Copy from previous version
   cp bin/php8.4.15/* bin/php8.4.16/
   ```

3. **Build the release** (fetch is automatic):
   ```bash
   gradle release -PbundleVersion=8.4.16
   ```
   
   The `release` task automatically runs `fetch` which will:
   - ✅ Update `build.properties` with today's date
   - ✅ Fetch the latest release tag from `modules-untouched` repository
   - ✅ Find and update ImageMagick, php_imagick, memcache, xdebug, and PEAR URLs
   - ✅ Update `bearsampp.conf` with the new PHP version
   - ✅ Update `deps.properties`, `exts.properties`, and `pear.properties`
   - ✅ Update version-specific paths in `php.ini`
   - ✅ Download PHP binaries and dependencies
   - ✅ Build and package the release

4. **Review the updated files** in `bin/php8.4.16/`:
   - `bearsampp.conf` - PHP version updated
   - `exts.properties` - Extension URLs updated
   - `deps.properties` - Dependency URLs updated
   - `pear.properties` - PEAR URL updated
   - `php.ini` - Version-specific paths updated

5. **Commit the changes** to git:
   ```bash
   git add bin/php8.4.16/
   git commit -m "Add PHP 8.4.16 configuration"
   ```

> **Note**: The `fetch` task is automatically included in the `release` build process. You only need to run `gradle fetch` separately if you want to update dependency URLs without building a release.

### How the Fetch Task Works

The `fetch` task integrates with the [Bearsampp modules-untouched](https://github.com/Bearsampp/modules-untouched) repository:

1. **Reads `php.properties`** from modules-untouched to get the PHP download URL
2. **Extracts the release tag** (e.g., `php-2025.12.7`) from the URL
3. **Queries GitHub API** to get all assets from that release
4. **Pattern matches** to find the latest versions of:
   - ImageMagick (portable Q16-HDRI x64)
   - php_imagick (matching PHP major.minor version)
   - php_memcache (matching PHP major.minor version)
   - php_xdebug (matching PHP major.minor version)
   - PEAR (pearweb_phars)
5. **Updates configuration files** with the new URLs
6. **Updates version-specific paths** in php.ini

This ensures all dependencies are automatically synchronized with the latest compatible versions from the modules-untouched releases.

### Prerequisites

| Requirement       | Version       | Purpose                                  |
|-------------------|---------------|------------------------------------------|
| **Java**          | 8+            | Required for Gradle execution            |
| **Gradle**        | 8.0+          | Build automation tool                    |
| **PowerShell**    | 5.0+          | DLL architecture validation              |
| **7-Zip**         | Latest        | Archive extraction (optional)            |

### Available Tasks

| Task                  | Description                                      |
|-----------------------|--------------------------------------------------|
| `release`             | Build release package (interactive/non-interactive) |
| `fetch`               | Fetch and update dependencies from modules-untouched |
| `clean`               | Clean build artifacts and temporary files        |
| `verify`              | Verify build environment and dependencies        |
| `info`                | Display build configuration information          |
| `listVersions`        | List available bundle versions in bin/           |
| `listExtensions`      | List PHP extensions configured in bin/           |
| `validateProperties`  | Validate build.properties configuration          |

For complete documentation, see [.gradle-docs/README.md](.gradle-docs/README.md)

## Documentation

- **Build Documentation**: [.gradle-docs/README.md](.gradle-docs/README.md)
- **Task Reference**: [.gradle-docs/TASKS.md](.gradle-docs/TASKS.md)
- **Configuration Guide**: [.gradle-docs/CONFIGURATION.md](.gradle-docs/CONFIGURATION.md)
- **API Reference**: [.gradle-docs/API.md](.gradle-docs/API.md)
- **CI/CD Testing**: [docs/PHP-EXTENSION-TESTING.md](docs/PHP-EXTENSION-TESTING.md)
- **Module Downloads**: https://bearsampp.com/module/php

## Issues

Issues must be reported on [Bearsampp repository](https://github.com/bearsampp/bearsampp/issues).

## Statistics

![Alt](https://repobeats.axiom.co/api/embed/2b56dc0b1aac6a6280b8051a41421d4fbb89ef49.svg "Repobeats analytics image")
