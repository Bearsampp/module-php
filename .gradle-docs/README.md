# Bearsampp Module PHP - Gradle Build Documentation

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Build Tasks](#build-tasks)
- [Configuration](#configuration)
- [Architecture](#architecture)
- [Troubleshooting](#troubleshooting)
- [Migration Guide](#migration-guide)

---

## Overview

The Bearsampp Module PHP project has been converted to a **pure Gradle build system**, replacing the legacy Ant build configuration. This provides:

- **Modern Build System**     - Native Gradle tasks and conventions
- **Better Performance**       - Incremental builds and caching
- **Simplified Maintenance**   - Pure Groovy/Gradle DSL
- **Enhanced Tooling**         - IDE integration and dependency management
- **Cross-Platform Support**   - Works on Windows, Linux, and macOS

### Project Information

| Property          | Value                                    |
|-------------------|------------------------------------------|
| **Project Name**  | module-php                               |
| **Group**         | com.bearsampp.modules                    |
| **Type**          | PHP Module Builder                       |
| **Build Tool**    | Gradle 8.x+                              |
| **Language**      | Groovy (Gradle DSL)                      |

---

## Quick Start

### Prerequisites

| Requirement       | Version       | Purpose                                  |
|-------------------|---------------|------------------------------------------|
| **Java**          | 8+            | Required for Gradle execution            |
| **Gradle**        | 8.0+          | Build automation tool                    |
| **PowerShell**    | 5.0+          | DLL architecture validation              |
| **7-Zip**         | Latest        | Archive extraction (optional)            |

### Basic Commands

```bash
# Display build information
gradle info

# List all available tasks
gradle tasks

# Verify build environment
gradle verify

# Build a release (interactive)
gradle release

# Build a specific version (non-interactive)
gradle release -PbundleVersion=8.3.15

# Clean build artifacts
gradle clean
```

---

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/bearsampp/module-php.git
cd module-php
```

### 2. Verify Environment

```bash
gradle verify
```

This will check:
- Java version (8+)
- Required files (build.properties, releases.properties)
- Directory structure (bin/, pear/)
- Build dependencies

### 3. List Available Versions

```bash
gradle listVersions
```

### 4. Build Your First Release

```bash
# Interactive mode (prompts for version)
gradle release

# Or specify version directly
gradle release -PbundleVersion=8.4.14
```

---

## Build Tasks

### Core Build Tasks

| Task                  | Description                                      | Example                                  |
|-----------------------|--------------------------------------------------|------------------------------------------|
| `release`             | Build and package release (interactive/non-interactive) | `gradle release -PbundleVersion=8.3.15` |
| `releaseBuild`        | Execute the release build process                | `gradle releaseBuild`                    |
| `clean`               | Clean build artifacts and temporary files        | `gradle clean`                           |

### Verification Tasks

| Task                      | Description                                  | Example                                      |
|---------------------------|----------------------------------------------|----------------------------------------------|
| `verify`                  | Verify build environment and dependencies    | `gradle verify`                              |
| `validateProperties`      | Validate build.properties configuration      | `gradle validateProperties`                  |
| `validateDllArchitecture` | Validate DLL architecture (64-bit check)     | `gradle validateDllArchitecture -PdllFile=<path>` |

### Information Tasks

| Task                | Description                                      | Example                    |
|---------------------|--------------------------------------------------|----------------------------|
| `info`              | Display build configuration information          | `gradle info`              |
| `phpInfo`           | Display PHP-specific build information           | `gradle phpInfo`           |
| `listVersions`      | List available bundle versions in bin/           | `gradle listVersions`      |
| `listReleases`      | List all available releases from releases.properties | `gradle listReleases`  |
| `listExtensions`    | List PHP extensions configured in bin/           | `gradle listExtensions`    |
| `listPearConfig`    | List PEAR configurations in bin/                 | `gradle listPearConfig`    |
| `listDependencies`  | List dependencies configured in bin/             | `gradle listDependencies`  |

### Task Groups

| Group            | Purpose                                          |
|------------------|--------------------------------------------------|
| **build**        | Build and package tasks                          |
| **verification** | Verification and validation tasks                |
| **help**         | Help and information tasks                       |

---

## Configuration

### build.properties

The main configuration file for the build:

```properties
bundle.name     = php
bundle.release  = 2025.10.31
bundle.type     = bins
bundle.format   = 7z
```

| Property          | Description                          | Example Value  |
|-------------------|--------------------------------------|----------------|
| `bundle.name`     | Name of the bundle                   | `php`          |
| `bundle.release`  | Release version                      | `2025.10.31`   |
| `bundle.type`     | Type of bundle                       | `bins`         |
| `bundle.format`   | Archive format                       | `7z`           |

### gradle.properties

Gradle-specific configuration:

```properties
# Gradle daemon configuration
org.gradle.daemon=true
org.gradle.parallel=true
org.gradle.caching=true

# JVM settings
org.gradle.jvmargs=-Xmx2g -XX:MaxMetaspaceSize=512m
```

### Directory Structure

```
module-php/
├── .gradle-docs/          # Gradle documentation
│   ├── README.md          # Main documentation
│   ├── TASKS.md           # Task reference
│   ├── CONFIGURATION.md   # Configuration guide
│   └── API.md             # API reference
├── bin/                   # PHP version bundles
│   ├── php8.3.15/
│   ├── php8.4.14/
│   └── ...
├── pear/                  # PEAR installation scripts
│   ├── pear-install.bat
│   └── pear-install.php
├── tmp/                   # Temporary build files
│   ├── prep/              # Prepared bundles
│   └── downloads/         # Downloaded dependencies
├── build/                 # Gradle build outputs
│   └── distributions/     # Final packaged archives (.7z/.zip)
├── build.gradle           # Main Gradle build script
├── settings.gradle        # Gradle settings
├── build.properties       # Build configuration
└── releases.properties    # Available PHP releases
```

---

## Architecture

### Build Process Flow

```
1. User runs: gradle release -PbundleVersion=8.3.15
                    ↓
2. Validate environment and version
                    ↓
3. Create preparation directory (tmp/prep/)
                    ↓
4. Copy base PHP files from bin/php8.3.15/
                    ↓
5. Process extensions (if exts.properties exists)
   - Download extensions
   - Validate 64-bit architecture
   - Copy to ext/ directory
   - Update php.ini
                    ↓
6. Process PEAR (if pear.properties exists)
   - Download PEAR phar
   - Execute installation
                    ↓
7. Process dependencies (if deps.properties exists)
   - Download dependencies (e.g., ImageMagick)
   - Copy to appropriate directories
                    ↓
8. Output prepared bundle to tmp/prep/
                    ↓
9. Package prepared folder into archive in build/distributions/
   - The archive includes the top-level folder: php{version}/
```

### Packaging Details

- Archive name format: `bearsampp-php-{version}-{bundle.release}.{7z|zip}`
- Location: `build/distributions/`
- Content root: the top-level folder inside the archive is `{bundle.release}/` (e.g., `2025.10.31/`).
- Inside `{bundle.release}/` the structure mirrors Bruno:
  - `bin/php{version}/...` → the prepared current version
  - `bin/archived/` → all archived versions from the repository’s `bin/archived/`
  - `releases.properties` (optional) at the same level as `bin/`

To verify the folder is included at the root of the archive:

```bash
# 7z
7z l build/distributions/bearsampp-php-8.4.14-2025.10.31.7z | more
# You should see entries beginning with:
#   2025.10.31/bin/php8.4.14/
#   2025.10.31/bin/archived/

# PowerShell (zip example)
Expand-Archive -Path build/distributions/bearsampp-php-8.4.14-2025.10.31.zip -DestinationPath .\_inspect
Get-ChildItem .\_inspect\2025.10.31\bin | Select-Object Name
```

### Extension Processing

Each PHP version can have an `exts.properties` file:

```properties
phpexts.imagick=https://windows.php.net/downloads/pecl/releases/imagick/3.7.0/php_imagick-3.7.0-8.3-ts-vs16-x64.zip
phpexts.redis=https://windows.php.net/downloads/pecl/releases/redis/6.0.2/php_redis-6.0.2-8.3-ts-vs16-x64.zip
phpexts.xdebug=https://xdebug.org/files/php_xdebug-3.3.1-8.3-vs16-x86_64.dll
```

### PEAR Processing

PEAR configuration in `pear.properties`:

```properties
phppear.pear=https://pear.php.net/install-pear-nozlib.phar
```

### Dependency Processing

Dependencies in `deps.properties`:

```properties
phpdeps.imagemagick=https://imagemagick.org/archive/binaries/ImageMagick-7.1.1-28-portable-Q16-HDRI-x64.zip
```

---

## Troubleshooting

### Common Issues

#### Issue: "Dev path not found"

**Symptom:**
```
Dev path not found: E:/Bearsampp-development/dev
```

**Solution:**
This is a warning only. The dev path is optional for most tasks. If you need it, ensure the `dev` project exists in the parent directory.

---

#### Issue: "Bundle version not found"

**Symptom:**
```
Bundle version not found: E:/Bearsampp-development/module-php/bin/php8.3.99
```

**Solution:**
1. List available versions: `gradle listVersions`
2. Use an existing version: `gradle release -PbundleVersion=8.3.15`

---

#### Issue: "DLL is not 64-bit architecture"

**Symptom:**
```
DLL php_extension.dll is not 64-bit architecture. Build halted.
```

**Solution:**
The extension DLL is not 64-bit compatible. Download the correct 64-bit version from the extension provider.

---

#### Issue: "Java version too old"

**Symptom:**
```
Java 8+ required
```

**Solution:**
1. Check Java version: `java -version`
2. Install Java 8 or higher
3. Update JAVA_HOME environment variable

---

### Debug Mode

Run Gradle with debug output:

```bash
gradle release -PbundleVersion=8.3.15 --info
gradle release -PbundleVersion=8.3.15 --debug
```

### Clean Build

If you encounter issues, try a clean build:

```bash
gradle clean
gradle release -PbundleVersion=8.3.15
```

---

## Migration Guide

### From Ant to Gradle

The project has been fully migrated from Ant to Gradle. Here's what changed:

#### Removed Files

| File              | Status    | Replacement                |
|-------------------|-----------|----------------------------|
| `build.xml`       | ❌ Removed | `build.gradle`             |
| `sigcheck.xml`    | ❌ Removed | Integrated into build.gradle |

#### Command Mapping

| Ant Command                          | Gradle Command                              |
|--------------------------------------|---------------------------------------------|
| `ant release`                        | `gradle release`                            |
| `ant release -Dinput.bundle=8.3.15`  | `gradle release -PbundleVersion=8.3.15`     |
| `ant clean`                          | `gradle clean`                              |

#### Key Differences

| Aspect              | Ant                          | Gradle                           |
|---------------------|------------------------------|----------------------------------|
| **Build File**      | XML (build.xml)              | Groovy DSL (build.gradle)        |
| **Task Definition** | `<target name="...">`        | `tasks.register('...')`          |
| **Properties**      | `<property name="..." />`    | `ext { ... }`                    |
| **Dependencies**    | Manual downloads             | Automatic with repositories      |
| **Caching**         | None                         | Built-in incremental builds      |
| **IDE Support**     | Limited                      | Excellent (IntelliJ, Eclipse)    |

---

## Additional Resources

- [Gradle Documentation](https://docs.gradle.org/)
- [Bearsampp Project](https://github.com/bearsampp/bearsampp)
- [PHP Downloads](https://windows.php.net/download/)
- [PECL Extensions](https://windows.php.net/downloads/pecl/)

---

## Support

For issues and questions:

- **GitHub Issues**: https://github.com/bearsampp/module-php/issues
- **Bearsampp Issues**: https://github.com/bearsampp/bearsampp/issues
- **Documentation**: https://bearsampp.com/module/php

---

**Last Updated**: 2025-01-31  
**Version**: 2025.10.31  
**Build System**: Pure Gradle (no wrapper, no Ant)

Notes:
- This project deliberately does not ship the Gradle Wrapper. Install Gradle 8+ locally and run with `gradle ...`.
- Legacy Ant files (e.g., Eclipse `.launch` referencing `build.xml`) are deprecated and not used by the build.
