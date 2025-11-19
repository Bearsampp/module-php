# Documentation Index

Complete index of all Gradle build documentation for Bearsampp Module PHP.

---

## Quick Links

| Document              | Description                                      | Link                          |
|-----------------------|--------------------------------------------------|-------------------------------|
| **Main Documentation**| Complete build system guide                      | [README.md](README.md)        |
| **Task Reference**    | All available Gradle tasks                       | [TASKS.md](TASKS.md)          |
| **Configuration**     | Configuration files and properties               | [CONFIGURATION.md](CONFIGURATION.md) |
| **API Reference**     | Build script API and helper functions            | [API.md](API.md)              |

---

## Documentation Structure

```
.gradle-docs/
├── INDEX.md              # This file - Documentation index
├── README.md             # Main documentation and quick start
├── TASKS.md              # Complete task reference
├── CONFIGURATION.md      # Configuration guide
└── API.md                # API reference for build scripts
```

---

## Getting Started

### New Users

1. **Start Here**: [README.md](README.md) - Overview and quick start
2. **Verify Setup**: Run `gradle verify` to check environment
3. **List Tasks**: Run `gradle tasks` to see available tasks
4. **Build Release**: Run `gradle release -PbundleVersion=8.3.15`

### Advanced Users

1. **Task Reference**: [TASKS.md](TASKS.md) - All tasks with examples
2. **Configuration**: [CONFIGURATION.md](CONFIGURATION.md) - Advanced configuration
3. **API Reference**: [API.md](API.md) - Build script API and extensions
4. **Custom Tasks**: Create custom tasks using API reference

---

## Documentation by Topic

### Build System

| Topic                 | Document              | Section                          |
|-----------------------|-----------------------|----------------------------------|
| Overview              | README.md             | Overview                         |
| Quick Start           | README.md             | Quick Start                      |
| Installation          | README.md             | Installation                     |
| Architecture          | README.md             | Architecture                     |

### Tasks

| Topic                 | Document              | Section                          |
|-----------------------|-----------------------|----------------------------------|
| Build Tasks           | TASKS.md              | Build Tasks                      |
| Verification Tasks    | TASKS.md              | Verification Tasks               |
| Information Tasks     | TASKS.md              | Information Tasks                |
| Task Examples         | TASKS.md              | Task Examples                    |

### Configuration

| Topic                 | Document              | Section                          |
|-----------------------|-----------------------|----------------------------------|
| Build Properties      | CONFIGURATION.md      | Build Properties                 |
| Gradle Properties     | CONFIGURATION.md      | Gradle Properties                |
| PHP Versions          | CONFIGURATION.md      | PHP Version Configuration        |
| Extensions            | CONFIGURATION.md      | Extension Configuration          |
| PEAR                  | CONFIGURATION.md      | PEAR Configuration               |
| Dependencies          | CONFIGURATION.md      | Dependency Configuration         |

### API

| Topic                 | Document              | Section                          |
|-----------------------|-----------------------|----------------------------------|
| Build Script API      | API.md                | Build Script API                 |
| Helper Functions      | API.md                | Helper Functions                 |
| Extension Points      | API.md                | Extension Points                 |
| Properties API        | API.md                | Properties API                   |
| Task API              | API.md                | Task API                         |

---

## Common Tasks

### Building

| Task                                      | Document      | Reference                        |
|-------------------------------------------|---------------|----------------------------------|
| Build a release                           | README.md     | Quick Start                      |
| Build specific version                    | TASKS.md      | release task                     |
| Clean build artifacts                     | TASKS.md      | clean task                       |

### Configuration

| Task                                      | Document      | Reference                        |
|-------------------------------------------|---------------|----------------------------------|
| Configure build properties                | CONFIGURATION.md | Build Properties              |
| Configure extensions                      | CONFIGURATION.md | Extension Configuration       |
| Configure PEAR                            | CONFIGURATION.md | PEAR Configuration            |
| Configure dependencies                    | CONFIGURATION.md | Dependency Configuration      |

### Verification

| Task                                      | Document      | Reference                        |
|-------------------------------------------|---------------|----------------------------------|
| Verify build environment                  | TASKS.md      | verify task                      |
| Validate properties                       | TASKS.md      | validateProperties task          |
| Validate DLL architecture                 | TASKS.md      | validateDllArchitecture task     |

### Information

| Task                                      | Document      | Reference                        |
|-------------------------------------------|---------------|----------------------------------|
| Display build info                        | TASKS.md      | info task                        |
| List available versions                   | TASKS.md      | listVersions task                |
| List extensions                           | TASKS.md      | listExtensions task              |
| List PEAR config                          | TASKS.md      | listPearConfig task              |

---

## Quick Reference

### Essential Commands

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

### Essential Files

| File                  | Purpose                                  |
|-----------------------|------------------------------------------|
| `build.gradle`        | Main Gradle build script                 |
| `settings.gradle`     | Gradle project settings                  |
| `build.properties`    | Build configuration                      |
| `gradle.properties`   | Gradle-specific settings                 |
| `releases.properties` | Available PHP releases                   |

### Essential Directories

| Directory             | Purpose                                  |
|-----------------------|------------------------------------------|
| `bin/`                | PHP version bundles                      |
| `pear/`               | PEAR installation scripts                |
| `bearsampp-build/tmp/` | Temporary build files (external)       |
| `bearsampp-build/bins/` | Final packaged archives (external)    |
| `.gradle-docs/`       | Gradle documentation                     |

---

## Search by Keyword

### A-C

| Keyword               | Document              | Section                          |
|-----------------------|-----------------------|----------------------------------|
| API                   | API.md                | All sections                     |
| Architecture          | README.md             | Architecture                     |
| Build                 | TASKS.md              | Build Tasks                      |
| Clean                 | TASKS.md              | clean task                       |
| Configuration         | CONFIGURATION.md      | All sections                     |

### D-F

| Keyword               | Document              | Section                          |
|-----------------------|-----------------------|----------------------------------|
| Dependencies          | CONFIGURATION.md      | Dependency Configuration         |
| DLL                   | TASKS.md              | validateDllArchitecture task     |
| Extensions            | CONFIGURATION.md      | Extension Configuration          |
| Files                 | CONFIGURATION.md      | Configuration Files              |

### G-M

| Keyword               | Document              | Section                          |
|-----------------------|-----------------------|----------------------------------|
| Gradle                | README.md             | All sections                     |
| Helper Functions      | API.md                | Helper Functions                 |
| Info                  | TASKS.md              | info task                        |
| Installation          | README.md             | Installation                     |

### P-R

| Keyword               | Document              | Section                          |
|-----------------------|-----------------------|----------------------------------|
| PEAR                  | CONFIGURATION.md      | PEAR Configuration               |
| Properties            | CONFIGURATION.md      | Build Properties                 |
| Release               | TASKS.md              | release task                     |

### S-Z

| Keyword               | Document              | Section                          |
|-----------------------|-----------------------|----------------------------------|
| Tasks                 | TASKS.md              | All sections                     |
| Troubleshooting       | README.md             | Troubleshooting                  |
| Validation            | TASKS.md              | Verification Tasks               |
| Verify                | TASKS.md              | verify task                      |

---

## Document Summaries

### README.md

**Purpose**: Main documentation and quick start guide

**Contents**:
- Overview of the Gradle build system
- Quick start guide with basic commands
- Installation instructions
- Complete task reference
- Configuration overview
- Architecture and build process flow
- Troubleshooting common issues
- Migration guide summary

**Target Audience**: All users, especially new users

---

### TASKS.md

**Purpose**: Complete reference for all Gradle tasks

**Contents**:
- Build tasks (release, releaseBuild, clean)
- Verification tasks (verify, validateProperties, validateDllArchitecture)
- Information tasks (info, phpInfo, listVersions, etc.)
- Task dependencies and execution order
- Task examples and usage patterns
- Task options and properties

**Target Audience**: Developers and build engineers

---

### CONFIGURATION.md

**Purpose**: Configuration guide for build system

**Contents**:
- Configuration files overview
- Build properties reference
- Gradle properties reference
- PHP version configuration
- Extension configuration
- PEAR configuration
- Dependency configuration
- Environment variables
- Configuration examples
- Best practices

**Target Audience**: Build engineers and advanced users

---

### API.md

**Purpose**: API reference for build scripts

**Contents**:
- Build script API
- Helper functions reference
- Extension points
- Properties API
- Task API
- File operations API
- Exec API
- Logger API
- Exception handling
- API examples

**Target Audience**: Advanced users and contributors

---

---

## Version History

| Version       | Date       | Changes                                  |
|---------------|------------|------------------------------------------|
| 2025.10.31    | 2025-01-31 | Initial Gradle documentation             |
|               |            | - Created README.md                      |
|               |            | - Created TASKS.md                       |
|               |            | - Created CONFIGURATION.md               |
|               |            | - Created API.md                         |
|               |            | - Created INDEX.md                       |

---

## Contributing

To contribute to the documentation:

1. **Fork Repository**: Fork the module-php repository
2. **Edit Documentation**: Make changes to documentation files
3. **Follow Style**: Maintain consistent formatting and style
4. **Test Examples**: Verify all code examples work
5. **Submit PR**: Create pull request with changes

### Documentation Style Guide

- Use Markdown formatting
- Include code examples
- Use tables for structured data
- Add links to related sections
- Keep language clear and concise
- Include practical examples

---

## Support

For documentation issues or questions:

- **GitHub Issues**: https://github.com/bearsampp/module-php/issues
- **Bearsampp Issues**: https://github.com/bearsampp/bearsampp/issues
- **Documentation**: This directory (.gradle-docs/)

---

**Last Updated**: 2025-01-31  
**Version**: 2025.10.31  
**Total Documents**: 5
