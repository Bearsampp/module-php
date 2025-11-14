# Migration Guide: Ant to Gradle

Complete guide for migrating from the legacy Ant build system to the new pure Gradle build.

---

## Table of Contents

- [Overview](#overview)
- [What Changed](#what-changed)
- [Command Mapping](#command-mapping)
- [File Changes](#file-changes)
- [Configuration Changes](#configuration-changes)
- [Task Equivalents](#task-equivalents)
- [Troubleshooting](#troubleshooting)

---

## Overview

The Bearsampp Module PHP project has been fully migrated from Apache Ant to Gradle. This migration provides:

- **Modern Build System**     - Native Gradle tasks and conventions
- **Better Performance**       - Incremental builds and caching
- **Simplified Maintenance**   - Pure Groovy/Gradle DSL
- **Enhanced Tooling**         - IDE integration and dependency management
- **Cross-Platform Support**   - Works on Windows, Linux, and macOS

### Migration Status

| Component             | Status        | Notes                                    |
|-----------------------|---------------|------------------------------------------|
| **Build Files**       | ✅ Complete   | Converted to build.gradle                |
| **Sigcheck**          | ✅ Complete   | Integrated into build.gradle             |
| **Release Process**   | ✅ Complete   | Native Gradle implementation             |
| **Extension Build**   | ✅ Complete   | Native Gradle implementation             |
| **PEAR Install**      | ✅ Complete   | Native Gradle implementation             |
| **Dependencies**      | ✅ Complete   | Native Gradle implementation             |
| **Documentation**     | ✅ Complete   | Comprehensive Gradle docs                |

---

## What Changed

### Removed Files

| File              | Status        | Replacement                              |
|-------------------|---------------|------------------------------------------|
| `build.xml`       | ❌ Removed    | `build.gradle`                           |
| `sigcheck.xml`    | ❌ Removed    | Integrated into `build.gradle`           |

### New Files

| File                          | Purpose                                  |
|-------------------------------|------------------------------------------|
| `build.gradle`                | Main Gradle build script (pure Gradle)   |
| `settings.gradle`             | Gradle project settings                  |
| `.gradle-docs/README.md`      | Main documentation                       |
| `.gradle-docs/TASKS.md`       | Task reference                           |
| `.gradle-docs/CONFIGURATION.md` | Configuration guide                    |
| `.gradle-docs/API.md`         | API reference                            |
| `.gradle-docs/MIGRATION.md`   | This migration guide                     |

### Modified Files

| File              | Changes                                  |
|-------------------|------------------------------------------|
| `README.md`       | Updated with Gradle build information    |
| `gradle.properties` | Enhanced with performance settings     |

---

## Command Mapping

### Basic Commands

| Ant Command                          | Gradle Command                              | Notes                    |
|--------------------------------------|---------------------------------------------|--------------------------|
| `ant`                                | `gradle info`                               | Default task changed     |
| `ant -projecthelp`                   | `gradle tasks`                              | List all tasks           |
| `ant release`                        | `gradle release`                            | Interactive mode         |
| `ant release -Dinput.bundle=8.3.15`  | `gradle release -PbundleVersion=8.3.15`     | Non-interactive mode     |
| `ant clean`                          | `gradle clean`                              | Clean artifacts          |

### Advanced Commands

| Ant Command                          | Gradle Command                              | Notes                    |
|--------------------------------------|---------------------------------------------|--------------------------|
| `ant -v release`                     | `gradle release --info`                     | Verbose output           |
| `ant -d release`                     | `gradle release --debug`                    | Debug output             |
| `ant -k release`                     | `gradle release --continue`                 | Continue on failure      |

---

## File Changes

### build.xml → build.gradle

#### Ant (build.xml)

```xml
<project name="module-php" basedir=".">
  <property name="project.basedir" location="." />
  <property name="bundle.name" value="php" />
  
  <target name="release.build">
    <echo message="Building release..." />
    <!-- Complex Ant logic -->
  </target>
</project>
```

#### Gradle (build.gradle)

```groovy
plugins {
    id 'base'
}

ext {
    projectBasedir = projectDir.absolutePath
    bundleName = buildProps.getProperty('bundle.name', 'php')
}

tasks.register('release') {
    group = 'build'
    description = 'Build release package'
    
    doLast {
        println "Building release..."
        // Native Gradle logic
    }
}
```

---

### sigcheck.xml → build.gradle

#### Ant (sigcheck.xml)

```xml
<project name="Sigcheck" default="executeSigcheck">
  <target name="executeSigcheck">
    <exec executable="powershell.exe">
      <!-- PowerShell command -->
    </exec>
  </target>
</project>
```

#### Gradle (build.gradle)

```groovy
def validateDllArchitecture(File dll) {
    def psCommand = """
    try {
        \$bytes = [System.IO.File]::ReadAllBytes('${dll.absolutePath}');
        // PowerShell logic
    } catch {
        Write-Output 'Error'
    }
    """.trim()
    
    exec {
        executable 'powershell.exe'
        args '-NoLogo', '-NoProfile', '-Command', psCommand
    }
}
```

---

## Configuration Changes

### Properties

#### Ant Properties

```xml
<property name="bundle.name" value="php" />
<property name="bundle.release" value="2025.10.31" />
<property file="build.properties" />
```

#### Gradle Properties

```groovy
def buildProps = new Properties()
file('build.properties').withInputStream { buildProps.load(it) }

ext {
    bundleName = buildProps.getProperty('bundle.name', 'php')
    bundleRelease = buildProps.getProperty('bundle.release', '1.0.0')
}
```

---

### Paths

#### Ant Paths

```xml
<property name="project.basedir" location="." />
<property name="root.dir" location="${project.basedir}/.." />
<property name="dev.path" location="${root.dir}/dev" />
```

#### Gradle Paths

```groovy
ext {
    projectBasedir = projectDir.absolutePath
    rootDir = projectDir.parent
    devPath = file("${rootDir}/dev").absolutePath
}
```

---

## Task Equivalents

### Release Task

#### Ant

```xml
<target name="release.build">
  <basename property="bundle.folder" file="${bundle.path}" />
  <getmoduleuntouched name="${bundle.name}" version="${bundle.version}" />
  <copy todir="${php.prep.path}">
    <fileset dir="${bundle.srcdest}" />
  </copy>
</target>
```

#### Gradle

```groovy
tasks.register('releaseBuild') {
    group = 'build'
    description = 'Execute the release build process'
    
    doLast {
        def bundlePath = file("${projectDir}/bin/${bundleName}${versionToBuild}")
        def phpPrepPath = file("${bundleTmpPrepPath}/${bundleFolder}")
        
        copy {
            from bundlePath
            into phpPrepPath
        }
    }
}
```

---

### Extension Processing

#### Ant

```xml
<for list="${phpexts}" param="phpext">
  <sequential>
    <download url="${phpexts.@{phpext}}" />
    <copy file="${phpext.dll}" tofile="${php.prep.path}/ext/php_@{phpext}.dll" />
  </sequential>
</for>
```

#### Gradle

```groovy
def processExtensions(File extsFile, File phpPrepPath) {
    def exts = new Properties()
    extsFile.withInputStream { exts.load(it) }
    
    exts.each { key, url ->
        def extName = key.toString().replace('phpexts.', '')
        def extArchive = downloadFile(url.toString(), downloadDir)
        // Process extension
    }
}
```

---

### PEAR Installation

#### Ant

```xml
<if>
  <available file="${php.prep.path}/pear.properties" type="file" />
  <then>
    <download url="${phppear.pear}" />
    <exec executable="${php.prep.path}/pear-install/pear-install.bat" />
  </then>
</if>
```

#### Gradle

```groovy
def processPear(File pearFile, File phpPrepPath) {
    def pear = new Properties()
    pearFile.withInputStream { pear.load(it) }
    
    def pearUrl = pear.getProperty('phppear.pear')
    def pearPhar = downloadFile(pearUrl, pearInstallDir)
    
    exec {
        executable 'cmd'
        args '/c', 'pear-install.bat'
        workingDir pearInstallDir
    }
}
```

---

## Troubleshooting

### Issue: "Cannot find Ant"

**Symptom:**
```
'ant' is not recognized as an internal or external command
```

**Solution:**
Ant is no longer needed. Use Gradle instead:
```bash
gradle release -PbundleVersion=8.3.15
```

---

### Issue: "build.xml not found"

**Symptom:**
```
build.xml not found in project directory
```

**Solution:**
The project now uses Gradle. Use `build.gradle` instead:
```bash
gradle tasks
gradle info
```

---

### Issue: "Old Ant scripts in documentation"

**Symptom:**
Documentation references Ant commands

**Solution:**
Refer to the new Gradle documentation:
- [.gradle-docs/README.md](.gradle-docs/README.md)
- [.gradle-docs/TASKS.md](.gradle-docs/TASKS.md)

---

### Issue: "Custom Ant targets missing"

**Symptom:**
Custom Ant targets are not available in Gradle

**Solution:**
Custom tasks can be added to `build.gradle`:
```groovy
tasks.register('customTask') {
    group = 'custom'
    description = 'Custom task'
    
    doLast {
        // Implementation
    }
}
```

---

## Benefits of Migration

### Performance Improvements

| Aspect                | Ant           | Gradle        | Improvement   |
|-----------------------|---------------|---------------|---------------|
| **Incremental Builds**| No            | Yes           | 50-90% faster |
| **Build Cache**       | No            | Yes           | Reuse outputs |
| **Parallel Execution**| Limited       | Yes           | Multi-core    |
| **Dependency Cache**  | No            | Yes           | Faster downloads |

---

### Developer Experience

| Aspect                | Ant           | Gradle        | Improvement   |
|-----------------------|---------------|---------------|---------------|
| **IDE Support**       | Limited       | Excellent     | IntelliJ, Eclipse, VS Code |
| **Task Discovery**    | Manual        | `gradle tasks`| Easy discovery |
| **Documentation**     | XML comments  | Comprehensive | Better docs   |
| **Debugging**         | Difficult     | Easy          | `--debug`, `--info` |

---

### Maintenance

| Aspect                | Ant           | Gradle        | Improvement   |
|-----------------------|---------------|---------------|---------------|
| **Build Logic**       | XML           | Groovy DSL    | More readable |
| **Extensibility**     | Limited       | Excellent     | Plugins, tasks |
| **Testing**           | Manual        | Built-in      | Test tasks    |
| **CI/CD**             | Custom        | Native        | Better integration |

---

## Next Steps

### For Developers

1. **Install Gradle**: Download from https://gradle.org/
2. **Verify Installation**: Run `gradle --version`
3. **Read Documentation**: See [.gradle-docs/README.md](.gradle-docs/README.md)
4. **Try Commands**: Run `gradle tasks` and `gradle info`
5. **Build Release**: Run `gradle release -PbundleVersion=8.3.15`

---

### For CI/CD

1. **Update Scripts**: Replace `ant` commands with `gradle`
2. **Install Gradle**: Add Gradle to CI/CD environment
3. **Update Paths**: Update artifact paths (now in `tmp/prep/`)
4. **Test Builds**: Verify builds work in CI/CD
5. **Update Documentation**: Update CI/CD documentation

---

### For Contributors

1. **Clone Repository**: `git clone https://github.com/bearsampp/module-php.git`
2. **Read Docs**: Review [.gradle-docs/README.md](.gradle-docs/README.md)
3. **Verify Environment**: Run `gradle verify`
4. **Make Changes**: Edit `build.gradle` or configuration files
5. **Test Changes**: Run `gradle release -PbundleVersion=8.3.15`
6. **Submit PR**: Create pull request with changes

---

## Support

For migration issues or questions:

- **GitHub Issues**: https://github.com/bearsampp/module-php/issues
- **Bearsampp Issues**: https://github.com/bearsampp/bearsampp/issues
- **Documentation**: [.gradle-docs/README.md](.gradle-docs/README.md)

---

**Last Updated**: 2025-01-31  
**Version**: 2025.10.31  
**Migration Status**: Complete ✅
