# Gradle Tasks Reference

Complete reference for all available Gradle tasks in the Bearsampp Module PHP project.

---

## Table of Contents

- [Build Tasks](#build-tasks)
- [Verification Tasks](#verification-tasks)
- [Information Tasks](#information-tasks)
- [Task Dependencies](#task-dependencies)
- [Task Examples](#task-examples)

---

## Build Tasks

### `release`

**Group:** build  
**Description:** Build release package (interactive or use -PbundleVersion=X.X.X for non-interactive)

**Usage:**
```bash
# Interactive mode (prompts for version)
gradle release

# Non-interactive mode (specify version)
gradle release -PbundleVersion=8.3.15
```

**Parameters:**

| Parameter         | Type     | Required | Description                    | Example      |
|-------------------|----------|----------|--------------------------------|--------------|
| `bundleVersion`   | String   | No       | PHP version to build           | `8.3.15`     |

**Process:**
1. Validates environment and version
2. Creates preparation directory
3. Copies base PHP files
4. Processes extensions (if configured)
5. Processes PEAR (if configured)
6. Processes dependencies (if configured)
7. Outputs prepared bundle

**Output Location:** `tmp/prep/php{version}/`

---

### `releaseBuild`

**Group:** build  
**Description:** Execute the release build process

**Usage:**
```bash
gradle releaseBuild -PbundleVersion=8.3.15
```

**Parameters:**

| Parameter         | Type     | Required | Description                    | Example      |
|-------------------|----------|----------|--------------------------------|--------------|
| `bundleVersion`   | String   | Yes      | PHP version to build           | `8.3.15`     |

**Note:** This task is typically called by the `release` task. Use `release` instead for normal builds.

---

### `clean`

**Group:** build  
**Description:** Clean build artifacts and temporary files

**Usage:**
```bash
gradle clean
```

**Cleans:**
- `build/` directory
- `tmp/` directory
- All temporary build files

**Output:**
```
Cleaned: E:/Bearsampp-development/module-php/build
Cleaned: E:/Bearsampp-development/module-php/tmp
[SUCCESS] Build artifacts cleaned
```

---

## Verification Tasks

### `verify`

**Group:** verification  
**Description:** Verify build environment and dependencies

**Usage:**
```bash
gradle verify
```

**Checks:**

| Check                  | Description                              | Required |
|------------------------|------------------------------------------|----------|
| Java 8+                | Java version 8 or higher                 | Yes      |
| build.properties       | Build configuration file exists          | Yes      |
| releases.properties    | Release definitions file exists          | Yes      |
| pear directory         | PEAR installation directory exists       | Yes      |
| pear-install.bat       | PEAR installation script exists          | Yes      |
| bin directory          | PHP versions directory exists            | Yes      |

**Output:**
```
Environment Check Results:
------------------------------------------------------------
  [PASS]     Java 8+
  [PASS]     build.properties
  [PASS]     releases.properties
  [PASS]     pear directory
  [PASS]     pear-install.bat
  [PASS]     bin directory
------------------------------------------------------------

[SUCCESS] All checks passed! Build environment is ready.
```

---

### `validateProperties`

**Group:** verification  
**Description:** Validate build.properties configuration

**Usage:**
```bash
gradle validateProperties
```

**Validates:**

| Property          | Required | Description                    |
|-------------------|----------|--------------------------------|
| `bundle.name`     | Yes      | Name of the bundle             |
| `bundle.release`  | Yes      | Release version                |
| `bundle.type`     | Yes      | Type of bundle                 |
| `bundle.format`   | Yes      | Archive format                 |

**Output:**
```
[SUCCESS] All required properties are present:
    bundle.name = php
    bundle.release = 2025.10.31
    bundle.type = bins
    bundle.format = 7z
```

---

### `validateDllArchitecture`

**Group:** verification  
**Description:** Validate DLL architecture (64-bit check)

**Usage:**
```bash
gradle validateDllArchitecture -PdllFile=path/to/file.dll
```

**Parameters:**

| Parameter    | Type     | Required | Description                    | Example                  |
|--------------|----------|----------|--------------------------------|--------------------------|
| `dllFile`    | String   | Yes      | Path to DLL file to validate   | `ext/php_redis.dll`      |

**Output:**
```
Validating architecture for: php_redis.dll
MachineType: 64-bit
[SUCCESS] DLL is 64-bit compatible
```

---

## Information Tasks

### `info`

**Group:** help  
**Description:** Display build configuration information

**Usage:**
```bash
gradle info
```

**Displays:**
- Project information (name, version, description)
- Bundle properties (name, release, type, format)
- Paths (project dir, root dir, dev path, PHP ext, PEAR install)
- Java information (version, home)
- Gradle information (version, home)
- Available task groups
- Quick start commands

---

### `phpInfo`

**Group:** help  
**Description:** Display PHP-specific build information

**Usage:**
```bash
gradle phpInfo
```

**Displays:**
- PHP extension processing information
- PEAR installation information
- Dependencies information
- Architecture verification information
- Configuration files
- Useful commands

---

### `listVersions`

**Group:** help  
**Description:** List all available bundle versions in bin/ directory

**Usage:**
```bash
gradle listVersions
```

**Output:**
```
Available php versions in bin/:
------------------------------------------------------------
  8.1.32
  8.1.33
  8.2.28
  8.2.29
  8.3.20
  8.3.24
  8.3.26
  8.3.27
  8.4.10
  8.4.11
  8.4.13
  8.4.14
------------------------------------------------------------
Total versions: 12

To build a specific version:
  gradle release -PbundleVersion=8.4.14
```

---

### `listReleases`

**Group:** help  
**Description:** List all available releases from releases.properties

**Usage:**
```bash
gradle listReleases
```

**Output:**
```
Available PHP Releases:
--------------------------------------------------------------------------------
  8.1.32     -> https://windows.php.net/downloads/releases/php-8.1.32-Win32-vs16-x64.zip
  8.1.33     -> https://windows.php.net/downloads/releases/php-8.1.33-Win32-vs16-x64.zip
  ...
--------------------------------------------------------------------------------
Total releases: 50
```

---

### `listExtensions`

**Group:** help  
**Description:** List PHP extensions configured in bin directories

**Usage:**
```bash
gradle listExtensions
```

**Output:**
```
Scanning for PHP extension configurations...
================================================================================

PHP 8.3.15:
--------------------------------------------------------------------------------
  imagick              -> https://windows.php.net/downloads/pecl/releases/imagick/3.7.0/php_imagick-3.7.0-8.3-ts-vs16-x64.zip
  redis                -> https://windows.php.net/downloads/pecl/releases/redis/6.0.2/php_redis-6.0.2-8.3-ts-vs16-x64.zip
  xdebug               -> https://xdebug.org/files/php_xdebug-3.3.1-8.3-vs16-x86_64.dll
================================================================================
```

---

### `listPearConfig`

**Group:** help  
**Description:** List PEAR configurations in bin directories

**Usage:**
```bash
gradle listPearConfig
```

**Output:**
```
Scanning for PEAR configurations...
================================================================================

PHP 8.3.15:
--------------------------------------------------------------------------------
  phppear.pear         -> https://pear.php.net/install-pear-nozlib.phar
================================================================================
```

---

### `listDependencies`

**Group:** help  
**Description:** List dependencies configured in bin directories

**Usage:**
```bash
gradle listDependencies
```

**Output:**
```
Scanning for dependency configurations...
================================================================================

PHP 8.3.15:
--------------------------------------------------------------------------------
  imagemagick          -> https://imagemagick.org/archive/binaries/ImageMagick-7.1.1-28-portable-Q16-HDRI-x64.zip
================================================================================
```

---

## Task Dependencies

### Task Execution Order

```
release
  └── releaseBuild
        ├── (validates environment)
        ├── (creates directories)
        ├── (copies base files)
        ├── (processes extensions)
        ├── (processes PEAR)
        └── (processes dependencies)
```

### Task Groups

| Group            | Tasks                                                                      |
|------------------|----------------------------------------------------------------------------|
| **build**        | `release`, `releaseBuild`, `clean`                                         |
| **verification** | `verify`, `validateProperties`, `validateDllArchitecture`                  |
| **help**         | `info`, `phpInfo`, `listVersions`, `listReleases`, `listExtensions`, `listPearConfig`, `listDependencies` |

---

## Task Examples

### Example 1: Complete Build Workflow

```bash
# 1. Verify environment
gradle verify

# 2. List available versions
gradle listVersions

# 3. Check extensions for a version
gradle listExtensions

# 4. Build the release
gradle release -PbundleVersion=8.3.15

# 5. Clean up
gradle clean
```

---

### Example 2: Debugging a Build

```bash
# Run with info logging
gradle release -PbundleVersion=8.3.15 --info

# Run with debug logging
gradle release -PbundleVersion=8.3.15 --debug

# Run with stack trace on error
gradle release -PbundleVersion=8.3.15 --stacktrace
```

---

### Example 3: Validation Workflow

```bash
# Validate build properties
gradle validateProperties

# Verify environment
gradle verify

# Validate a specific DLL
gradle validateDllArchitecture -PdllFile=ext/php_redis.dll
```

---

### Example 4: Information Gathering

```bash
# Get build info
gradle info

# Get PHP-specific info
gradle phpInfo

# List all available versions
gradle listVersions

# List all releases
gradle listReleases

# List extensions for all versions
gradle listExtensions

# List PEAR configs
gradle listPearConfig

# List dependencies
gradle listDependencies
```

---

## Task Options

### Common Gradle Options

| Option              | Description                              | Example                                  |
|---------------------|------------------------------------------|------------------------------------------|
| `--info`            | Set log level to INFO                    | `gradle release --info`                  |
| `--debug`           | Set log level to DEBUG                   | `gradle release --debug`                 |
| `--stacktrace`      | Print stack trace on error               | `gradle release --stacktrace`            |
| `--scan`            | Create build scan                        | `gradle release --scan`                  |
| `--dry-run`         | Show what would be executed              | `gradle release --dry-run`               |
| `--parallel`        | Execute tasks in parallel                | `gradle release --parallel`              |
| `--offline`         | Execute build without network access     | `gradle release --offline`               |

---

## Task Properties

### Project Properties

Set via `-P` flag:

| Property          | Type     | Description                    | Example                                  |
|-------------------|----------|--------------------------------|------------------------------------------|
| `bundleVersion`   | String   | PHP version to build           | `-PbundleVersion=8.3.15`                 |
| `dllFile`         | String   | DLL file to validate           | `-PdllFile=ext/php_redis.dll`            |

### System Properties

Set via `-D` flag:

| Property          | Type     | Description                    | Example                                  |
|-------------------|----------|--------------------------------|------------------------------------------|
| `org.gradle.daemon` | Boolean | Enable Gradle daemon         | `-Dorg.gradle.daemon=true`               |
| `org.gradle.parallel` | Boolean | Enable parallel execution  | `-Dorg.gradle.parallel=true`             |

---

**Last Updated**: 2025-01-31  
**Version**: 2025.10.31
