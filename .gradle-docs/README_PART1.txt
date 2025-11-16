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
â”œâ”€â”€ .gradle-docs/          # Gradle documentation
â”‚   â”œâ”€â”€ README.md          # Main documentation
â”‚   â”œâ”€â”€ TASKS.md           # Task reference
â”‚   â”œâ”€â”€ CONFIGURATION.md   # Configuration guide
â”‚   â””â”€â”€ API.md             # API reference
â”œâ”€â”€ bin/                   # PHP version bundles
â”‚   â”œâ”€â”€ php8.3.15/
â”‚   â”œâ”€â”€ php8.4.14/
â”‚   â””â”€â”€ ...
â”œâ”€â”€ pear/                  # PEAR installation scripts
â”‚   â”œâ”€â”€ pear-install.bat
â”‚   â””â”€â”€ pear-install.php
â”œâ”€â”€ tmp/                   # Temporary build files
â”‚   â”œâ”€â”€ prep/              # Prepared bundles
â”‚   â””â”€â”€ downloads/         # Downloaded dependencies
â”œâ”€â”€ build/                 # Gradle build outputs
â”‚   â””â”€â”€ distributions/     # Final packaged archives (.7z/.zip)
â”œâ”€â”€ build.gradle           # Main Gradle build script
â”œâ”€â”€ settings.gradle        # Gradle settings
â”œâ”€â”€ build.properties       # Build configuration
â””â”€â”€ releases.properties    # Available PHP releases
```

---

## Architecture

### Build Process Flow

```
1. User runs: gradle release -PbundleVersion=8.3.15
                    â†“
2. Validate environment and version
                    â†“
3. Create preparation directory (tmp/prep/)
                    â†“
4. Copy base PHP files from bin/php8.3.15/
                    â†“
5. Process extensions (if exts.properties exists)
   - Download extensions
   - Validate 64-bit architecture
   - Copy to ext/ directory
   - Update php.ini
                    â†“
6. Process PEAR (if pear.properties exists)
   - Download PEAR phar
   - Execute installation
                    â†“
7. Process dependencies (if deps.properties exists)
   - Download dependencies (e.g., ImageMagick)
   - Copy to appropriate directories
                    â†“
8. Output prepared bundle to tmp/prep/
                    â†“
9. Package prepared folder into archive in build/distributions/
   - The archive includes the top-level folder: php{version}/
```

