# Configuration Guide

Complete configuration guide for the Bearsampp Module PHP Gradle build system.

---

## Table of Contents

- [Configuration Files](#configuration-files)
- [Build Properties](#build-properties)
- [Gradle Properties](#gradle-properties)
- [PHP Version Configuration](#php-version-configuration)
- [Extension Configuration](#extension-configuration)
- [PEAR Configuration](#pear-configuration)
- [Dependency Configuration](#dependency-configuration)
- [Environment Variables](#environment-variables)

---

## Configuration Files

### Overview

| File                  | Purpose                                  | Format     | Location      |
|-----------------------|------------------------------------------|------------|---------------|
| `build.properties`    | Main build configuration                 | Properties | Root          |
| `gradle.properties`   | Gradle-specific settings                 | Properties | Root          |
| `settings.gradle`     | Gradle project settings                  | Groovy     | Root          |
| `build.gradle`        | Main build script                        | Groovy     | Root          |
| `releases.properties` | Available PHP releases                   | Properties | Root          |
| `exts.properties`     | PHP extensions per version               | Properties | bin/php{ver}/ |
| `pear.properties`     | PEAR configuration per version           | Properties | bin/php{ver}/ |
| `deps.properties`     | Dependencies per version                 | Properties | bin/php{ver}/ |

---

## Build Properties

### File: `build.properties`

**Location:** `E:/Bearsampp-development/module-php/build.properties`

**Purpose:** Main build configuration for the PHP module

### Properties

| Property          | Type     | Required | Default      | Description                          |
|-------------------|----------|----------|--------------|--------------------------------------|
| `bundle.name`     | String   | Yes      | `php`        | Name of the bundle                   |
| `bundle.release`  | String   | Yes      | -            | Release version (YYYY.MM.DD)         |
| `bundle.type`     | String   | Yes      | `bins`       | Type of bundle                       |
| `bundle.format`   | String   | Yes      | `7z`         | Archive format for output            |

### Example

```properties
bundle.name     = php
bundle.release  = 2025.10.31
bundle.type     = bins
bundle.format   = 7z
```

### Usage in Build Script

```groovy
def buildProps = new Properties()
file('build.properties').withInputStream { buildProps.load(it) }

def bundleName = buildProps.getProperty('bundle.name', 'php')
def bundleRelease = buildProps.getProperty('bundle.release', '1.0.0')
```

---

## Gradle Properties

### File: `gradle.properties`

**Location:** `E:/Bearsampp-development/module-php/gradle.properties`

**Purpose:** Gradle-specific configuration and JVM settings

### Properties

| Property                      | Type     | Default      | Description                          |
|-------------------------------|----------|--------------|--------------------------------------|
| `org.gradle.daemon`           | Boolean  | `true`       | Enable Gradle daemon                 |
| `org.gradle.parallel`         | Boolean  | `true`       | Enable parallel task execution       |
| `org.gradle.caching`          | Boolean  | `true`       | Enable build caching                 |
| `org.gradle.jvmargs`          | String   | -            | JVM arguments for Gradle             |
| `org.gradle.configureondemand`| Boolean  | `false`      | Configure projects on demand         |

### Example

```properties
# Gradle daemon configuration
org.gradle.daemon=true
org.gradle.parallel=true
org.gradle.caching=true

# JVM settings
org.gradle.jvmargs=-Xmx2g -XX:MaxMetaspaceSize=512m -XX:+HeapDumpOnOutOfMemoryError

# Configure on demand
org.gradle.configureondemand=false
```

### Performance Tuning

| Setting                       | Recommended Value | Purpose                              |
|-------------------------------|-------------------|--------------------------------------|
| `org.gradle.daemon`           | `true`            | Faster builds with daemon            |
| `org.gradle.parallel`         | `true`            | Parallel task execution              |
| `org.gradle.caching`          | `true`            | Cache task outputs                   |
| `org.gradle.jvmargs`          | `-Xmx2g`          | Allocate 2GB heap for Gradle         |

---

## PHP Version Configuration

### Directory Structure

Each PHP version has its own directory in `bin/`:

```
bin/
├── php8.1.32/
│   ├── exts.properties
│   ├── pear.properties
│   ├── deps.properties
│   └── php.ini
├── php8.3.15/
│   ├── exts.properties
│   ├── pear.properties
│   ├── deps.properties
│   └── php.ini
└── php8.4.14/
    ├── exts.properties
    ├── pear.properties
    ├── deps.properties
    └── php.ini
```

### Version Naming Convention

| Pattern           | Example       | Description                          |
|-------------------|---------------|--------------------------------------|
| `php{major}.{minor}.{patch}` | `php8.3.15` | Standard PHP version format |

---

## Extension Configuration

### File: `exts.properties`

**Location:** `bin/php{version}/exts.properties`

**Purpose:** Define PHP extensions to include in the build

### Format

```properties
phpexts.{extension_name}={download_url}
```

### Example

```properties
phpexts.imagick=https://windows.php.net/downloads/pecl/releases/imagick/3.7.0/php_imagick-3.7.0-8.3-ts-vs16-x64.zip
phpexts.redis=https://windows.php.net/downloads/pecl/releases/redis/6.0.2/php_redis-6.0.2-8.3-ts-vs16-x64.zip
phpexts.xdebug=https://xdebug.org/files/php_xdebug-3.3.1-8.3-vs16-x86_64.dll
phpexts.opcache=https://windows.php.net/downloads/pecl/releases/opcache/8.3.15/php_opcache-8.3.15-ts-vs16-x64.zip
```

### Extension Properties

| Property                  | Type     | Required | Description                          |
|---------------------------|----------|----------|--------------------------------------|
| `phpexts.{name}`          | URL      | Yes      | Download URL for extension           |

### Supported Extension Formats

| Format    | Description                              | Example                              |
|-----------|------------------------------------------|--------------------------------------|
| `.zip`    | ZIP archive containing DLL               | `php_redis-6.0.2-8.3-ts-vs16-x64.zip` |
| `.dll`    | Direct DLL file                          | `php_xdebug-3.3.1-8.3-vs16-x86_64.dll` |

### Extension Processing

1. **Download:** Extension is downloaded from the specified URL
2. **Extract:** If ZIP, extract to temporary directory
3. **Validate:** Check DLL is 64-bit architecture
4. **Copy:** Copy `php_{name}.dll` to `ext/` directory
5. **Configure:** Add `extension={name}` to `php.ini` (except xdebug)

### Special Extensions

| Extension     | Special Handling                         |
|---------------|------------------------------------------|
| `imagick`     | Copies `CORE_*.dll` files to `imagick/` directory |
| `xdebug`      | Not added to `extension=` list in php.ini |

---

## PEAR Configuration

### File: `pear.properties`

**Location:** `bin/php{version}/pear.properties`

**Purpose:** Configure PEAR installation for the PHP version

### Format

```properties
phppear.pear={pear_phar_url}
```

### Example

```properties
phppear.pear=https://pear.php.net/install-pear-nozlib.phar
```

### PEAR Properties

| Property          | Type     | Required | Description                          |
|-------------------|----------|----------|--------------------------------------|
| `phppear.pear`    | URL      | Yes      | URL to PEAR installation phar        |

### PEAR Processing

1. **Copy Scripts:** Copy `pear-install.bat` and `pear-install.php` to temp directory
2. **Download:** Download PEAR phar from specified URL
3. **Rename:** Rename to `install-pear-nozlib.phar`
4. **Execute:** Run `pear-install.bat` to install PEAR
5. **Cleanup:** Remove temporary installation files

---

## Dependency Configuration

### File: `deps.properties`

**Location:** `bin/php{version}/deps.properties`

**Purpose:** Define external dependencies required by PHP extensions

### Format

```properties
phpdeps.{dependency_name}={download_url}
```

### Example

```properties
phpdeps.imagemagick=https://imagemagick.org/archive/binaries/ImageMagick-7.1.1-28-portable-Q16-HDRI-x64.zip
```

### Dependency Properties

| Property              | Type     | Required | Description                          |
|-----------------------|----------|----------|--------------------------------------|
| `phpdeps.{name}`      | URL      | Yes      | Download URL for dependency          |

### Supported Dependencies

| Dependency        | Purpose                              | Target Directory |
|-------------------|--------------------------------------|------------------|
| `imagemagick`     | Image processing for imagick ext     | `imagick/`       |

### Dependency Processing

1. **Download:** Dependency is downloaded from the specified URL
2. **Extract:** If archive, extract to temporary directory
3. **Copy:** Copy required files to target directory
4. **Validate:** Ensure all required files are present

---

## Environment Variables

### Build Environment

| Variable          | Description                          | Example                              |
|-------------------|--------------------------------------|--------------------------------------|
| `JAVA_HOME`       | Java installation directory          | `C:\Program Files\Java\jdk-11`       |
| `GRADLE_HOME`     | Gradle installation directory        | `C:\Gradle\gradle-8.5`               |
| `PATH`            | System path (includes Java, Gradle)  | -                                    |

### Optional Variables

| Variable          | Description                          | Default                              |
|-------------------|--------------------------------------|--------------------------------------|
| `GRADLE_USER_HOME`| Gradle user home directory           | `~/.gradle`                          |
| `GRADLE_OPTS`     | Additional Gradle JVM options        | -                                    |

---

## Configuration Examples

### Example 1: Basic PHP 8.3.15 Configuration

**build.properties:**
```properties
bundle.name     = php
bundle.release  = 2025.10.31
bundle.type     = bins
bundle.format   = 7z
```

**bin/php8.3.15/exts.properties:**
```properties
phpexts.redis=https://windows.php.net/downloads/pecl/releases/redis/6.0.2/php_redis-6.0.2-8.3-ts-vs16-x64.zip
phpexts.xdebug=https://xdebug.org/files/php_xdebug-3.3.1-8.3-vs16-x86_64.dll
```

**bin/php8.3.15/pear.properties:**
```properties
phppear.pear=https://pear.php.net/install-pear-nozlib.phar
```

---

### Example 2: PHP 8.4.14 with ImageMagick

**bin/php8.4.14/exts.properties:**
```properties
phpexts.imagick=https://windows.php.net/downloads/pecl/releases/imagick/3.7.0/php_imagick-3.7.0-8.4-ts-vs16-x64.zip
phpexts.redis=https://windows.php.net/downloads/pecl/releases/redis/6.0.2/php_redis-6.0.2-8.4-ts-vs16-x64.zip
```

**bin/php8.4.14/deps.properties:**
```properties
phpdeps.imagemagick=https://imagemagick.org/archive/binaries/ImageMagick-7.1.1-28-portable-Q16-HDRI-x64.zip
```

---

### Example 3: Minimal Configuration (No Extensions)

**bin/php8.1.32/exts.properties:**
```properties
# No extensions configured
```

**bin/php8.1.32/pear.properties:**
```properties
phppear.pear=https://pear.php.net/install-pear-nozlib.phar
```

---

## Configuration Validation

### Validate Configuration

```bash
# Validate build.properties
gradle validateProperties

# Verify entire environment
gradle verify

# List configured extensions
gradle listExtensions

# List configured PEAR
gradle listPearConfig

# List configured dependencies
gradle listDependencies
```

### Validation Checklist

| Item                      | Command                          | Expected Result              |
|---------------------------|----------------------------------|------------------------------|
| Build properties          | `gradle validateProperties`      | All required properties set  |
| Environment               | `gradle verify`                  | All checks pass              |
| Extensions                | `gradle listExtensions`          | Extensions listed            |
| PEAR                      | `gradle listPearConfig`          | PEAR config listed           |
| Dependencies              | `gradle listDependencies`        | Dependencies listed          |

---

## Best Practices

### Configuration Management

1. **Version Control:** Keep all `.properties` files in version control
2. **Documentation:** Document custom configurations
3. **Validation:** Always run `gradle verify` after configuration changes
4. **Testing:** Test builds with new configurations before committing
5. **Backup:** Keep backups of working configurations

### Extension Management

1. **64-bit Only:** Always use 64-bit extensions
2. **Version Matching:** Match extension version to PHP version
3. **Thread Safety:** Use thread-safe (TS) extensions
4. **Compiler:** Match Visual Studio compiler version (vs16)
5. **Testing:** Test extensions after adding to configuration

### Performance Optimization

1. **Gradle Daemon:** Enable for faster builds
2. **Parallel Execution:** Enable for multi-core systems
3. **Build Cache:** Enable for incremental builds
4. **JVM Heap:** Allocate sufficient memory (2GB+)
5. **Network:** Use fast, reliable network for downloads

---

**Last Updated**: 2025-01-31  
**Version**: 2025.10.31
