# API Reference

Complete API reference for the Bearsampp Module PHP Gradle build system.

---

## Table of Contents

- [Build Script API](#build-script-api)
- [Helper Functions](#helper-functions)
- [Extension Points](#extension-points)
- [Properties API](#properties-api)
- [Task API](#task-api)

---

## Build Script API

### Project Configuration

#### `group`

**Type:** `String`  
**Default:** `com.bearsampp.modules`  
**Description:** Maven group ID for the project

```groovy
group = 'com.bearsampp.modules'
```

---

#### `version`

**Type:** `String`  
**Default:** Value from `build.properties`  
**Description:** Project version

```groovy
version = buildProps.getProperty('bundle.release', '1.0.0')
```

---

#### `description`

**Type:** `String`  
**Default:** Generated from bundle name  
**Description:** Project description

```groovy
description = "Bearsampp Module - ${buildProps.getProperty('bundle.name', 'php')}"
```

---

### Extension Properties

#### `ext.projectBasedir`

**Type:** `String`  
**Description:** Absolute path to project directory

```groovy
ext.projectBasedir = projectDir.absolutePath
```

---

#### `ext.rootDir`

**Type:** `String`  
**Description:** Absolute path to parent directory

```groovy
ext.rootDir = projectDir.parent
```

---

#### `ext.devPath`

**Type:** `String`  
**Description:** Absolute path to dev directory

```groovy
ext.devPath = file("${rootDir}/dev").absolutePath
```

---

#### `ext.bundleName`

**Type:** `String`  
**Default:** `php`  
**Description:** Name of the bundle from build.properties

```groovy
ext.bundleName = buildProps.getProperty('bundle.name', 'php')
```

---

#### `ext.bundleRelease`

**Type:** `String`  
**Default:** `1.0.0`  
**Description:** Release version from build.properties

```groovy
ext.bundleRelease = buildProps.getProperty('bundle.release', '1.0.0')
```

---

#### `ext.bundleType`

**Type:** `String`  
**Default:** `bins`  
**Description:** Bundle type from build.properties

```groovy
ext.bundleType = buildProps.getProperty('bundle.type', 'bins')
```

---

#### `ext.bundleFormat`

**Type:** `String`  
**Default:** `7z`  
**Description:** Archive format from build.properties

```groovy
ext.bundleFormat = buildProps.getProperty('bundle.format', '7z')
```

---

#### `ext.phpExtPath`

**Type:** `String`  
**Description:** Absolute path to PHP extensions directory

```groovy
ext.phpExtPath = file("${projectDir}/ext").absolutePath
```

---

#### `ext.pearInstallPath`

**Type:** `String`  
**Description:** Absolute path to PEAR installation directory

```groovy
ext.pearInstallPath = file("${projectDir}/pear").absolutePath
```

---

#### `ext.bundleTmpPrepPath`

**Type:** `String`  
**Description:** Absolute path to temporary preparation directory

```groovy
ext.bundleTmpPrepPath = file("${projectDir}/tmp/prep").absolutePath
```

---

#### `ext.buildTmpPath`

**Type:** `String`  
**Description:** Absolute path to temporary build directory

```groovy
ext.buildTmpPath = file("${projectDir}/tmp").absolutePath
```

---

## Helper Functions

### `downloadFile(String url, File destDir)`

**Description:** Download a file from URL to destination directory

**Parameters:**

| Parameter    | Type     | Description                          |
|--------------|----------|--------------------------------------|
| `url`        | String   | URL to download from                 |
| `destDir`    | File     | Destination directory                |

**Returns:** `File` - Downloaded file

**Example:**

```groovy
def archive = downloadFile(
    'https://example.com/file.zip',
    file("${buildTmpPath}/downloads")
)
```

**Process:**
1. Extract filename from URL
2. Check if file already exists
3. If not exists, download using URL.withInputStream
4. Return File object

---

### `extractArchive(File archive, File destDir)`

**Description:** Extract archive (ZIP or 7z) to destination directory

**Parameters:**

| Parameter    | Type     | Description                          |
|--------------|----------|--------------------------------------|
| `archive`    | File     | Archive file to extract              |
| `destDir`    | File     | Destination directory                |

**Returns:** `void`

**Example:**

```groovy
extractArchive(
    file("${buildTmpPath}/downloads/extension.zip"),
    file("${buildTmpPath}/extracted")
)
```

**Supported Formats:**
- `.zip` - Extracted using Gradle's `zipTree()`
- `.7z` - Extracted using 7-Zip command line

---

### `validateDllArchitecture(File dll)`

**Description:** Validate DLL is 64-bit architecture

**Parameters:**

| Parameter    | Type     | Description                          |
|--------------|----------|--------------------------------------|
| `dll`        | File     | DLL file to validate                 |

**Returns:** `void`

**Throws:** `GradleException` if DLL is not 64-bit

**Example:**

```groovy
validateDllArchitecture(file('ext/php_redis.dll'))
```

**Process:**
1. Read DLL file bytes
2. Parse PE header
3. Check machine type (0x8664 = 64-bit)
4. Throw exception if not 64-bit

---

### `processExtensions(File extsFile, File phpPrepPath)`

**Description:** Process PHP extensions from configuration file

**Parameters:**

| Parameter      | Type     | Description                          |
|----------------|----------|--------------------------------------|
| `extsFile`     | File     | exts.properties file                 |
| `phpPrepPath`  | File     | PHP preparation directory            |

**Returns:** `void`

**Example:**

```groovy
def extsFile = new File(bundlePath, 'exts.properties')
if (extsFile.exists()) {
    processExtensions(extsFile, phpPrepPath)
}
```

**Process:**
1. Load extensions from properties file
2. For each extension:
   - Download extension archive/DLL
   - Extract if needed
   - Find main DLL file
   - Validate 64-bit architecture
   - Copy to ext/ directory
   - Handle special cases (imagick)
   - Add to php.ini (except xdebug)

---

### `processPear(File pearFile, File phpPrepPath)`

**Description:** Process PEAR installation from configuration file

**Parameters:**

| Parameter      | Type     | Description                          |
|----------------|----------|--------------------------------------|
| `pearFile`     | File     | pear.properties file                 |
| `phpPrepPath`  | File     | PHP preparation directory            |

**Returns:** `void`

**Example:**

```groovy
def pearFile = new File(bundlePath, 'pear.properties')
if (pearFile.exists()) {
    processPear(pearFile, phpPrepPath)
}
```

**Process:**
1. Load PEAR configuration
2. Copy pear-install scripts
3. Download PEAR phar
4. Execute installation
5. Cleanup temporary files

---

### `processDependencies(File depsFile, File phpPrepPath)`

**Description:** Process dependencies from configuration file

**Parameters:**

| Parameter      | Type     | Description                          |
|----------------|----------|--------------------------------------|
| `depsFile`     | File     | deps.properties file                 |
| `phpPrepPath`  | File     | PHP preparation directory            |

**Returns:** `void`

**Example:**

```groovy
def depsFile = new File(bundlePath, 'deps.properties')
if (depsFile.exists()) {
    processDependencies(depsFile, phpPrepPath)
}
```

**Process:**
1. Load dependencies from properties file
2. For each dependency:
   - Download dependency archive
   - Extract if needed
   - Copy to appropriate directory
   - Handle special cases (imagemagick)

---

## Extension Points

### Custom Task Registration

Register custom tasks using Gradle's task API:

```groovy
tasks.register('customTask') {
    group = 'custom'
    description = 'Custom task description'
    
    doLast {
        // Task implementation
    }
}
```

---

### Custom Extension Processing

Override extension processing by defining custom function:

```groovy
def customProcessExtensions(File extsFile, File phpPrepPath) {
    // Custom implementation
}
```

---

### Custom Validation

Add custom validation tasks:

```groovy
tasks.register('customValidation') {
    group = 'verification'
    description = 'Custom validation'
    
    doLast {
        // Validation logic
    }
}
```

---

## Properties API

### Project Properties

Access via `project.findProperty()`:

```groovy
def bundleVersion = project.findProperty('bundleVersion')
def dllFile = project.findProperty('dllFile')
```

---

### Build Properties

Access via loaded Properties object:

```groovy
def buildProps = new Properties()
file('build.properties').withInputStream { buildProps.load(it) }

def bundleName = buildProps.getProperty('bundle.name')
def bundleRelease = buildProps.getProperty('bundle.release')
```

---

### System Properties

Access via `System.getProperty()`:

```groovy
def javaHome = System.getProperty('java.home')
def javaVersion = System.getProperty('java.version')
```

---

### Environment Variables

Access via `System.getenv()`:

```groovy
def gradleHome = System.getenv('GRADLE_HOME')
def path = System.getenv('PATH')
```

---

## Task API

### Task Registration

```groovy
tasks.register('taskName') {
    group = 'groupName'
    description = 'Task description'
    
    doFirst {
        // Executed first
    }
    
    doLast {
        // Executed last
    }
}
```

---

### Task Dependencies

```groovy
tasks.register('taskB') {
    dependsOn 'taskA'
    
    doLast {
        // Executed after taskA
    }
}
```

---

### Task Configuration

```groovy
tasks.named('existingTask') {
    // Configure existing task
    doLast {
        // Add additional action
    }
}
```

---

### Task Execution

```groovy
// Execute task programmatically
tasks.getByName('taskName').execute()

// Execute task actions
tasks.getByName('taskName').actions.each { action ->
    action.execute(tasks.getByName('taskName'))
}
```

---

## File API

### File Operations

```groovy
// Create file object
def file = file('path/to/file')

// Check existence
if (file.exists()) { }

// Create directory
file.mkdirs()

// Delete file/directory
delete file

// Copy files
copy {
    from 'source'
    into 'destination'
    include '*.txt'
    exclude '*.log'
}
```

---

### Archive Operations

```groovy
// Extract ZIP
copy {
    from zipTree('archive.zip')
    into 'destination'
}

// Extract 7z
exec {
    executable '7z'
    args 'x', 'archive.7z', '-odestination', '-y'
}
```

---

## Exec API

### Execute Command

```groovy
exec {
    executable 'command'
    args 'arg1', 'arg2'
    workingDir file('directory')
    standardOutput = new ByteArrayOutputStream()
    ignoreExitValue = true
}
```

---

### PowerShell Execution

```groovy
def output = new ByteArrayOutputStream()
exec {
    executable 'powershell.exe'
    args '-NoLogo', '-NoProfile', '-Command', 'Get-Date'
    standardOutput = output
}
def result = output.toString().trim()
```

---

## Logger API

### Logging Levels

```groovy
logger.error('Error message')
logger.warn('Warning message')
logger.lifecycle('Lifecycle message')
logger.quiet('Quiet message')
logger.info('Info message')
logger.debug('Debug message')
```

---

### Conditional Logging

```groovy
if (logger.isInfoEnabled()) {
    logger.info('Info message')
}

if (logger.isDebugEnabled()) {
    logger.debug('Debug message')
}
```

---

## Exception API

### Throw Exception

```groovy
throw new GradleException('Error message')
```

---

### Custom Exception

```groovy
class CustomException extends GradleException {
    CustomException(String message) {
        super(message)
    }
}

throw new CustomException('Custom error')
```

---

## Examples

### Example 1: Custom Download Task

```groovy
tasks.register('downloadPhp') {
    group = 'download'
    description = 'Download PHP version'
    
    doLast {
        def version = project.findProperty('phpVersion')
        if (!version) {
            throw new GradleException('phpVersion property required')
        }
        
        def url = "https://windows.php.net/downloads/releases/php-${version}-Win32-vs16-x64.zip"
        def destDir = file("${buildTmpPath}/downloads")
        
        def archive = downloadFile(url, destDir)
        println "Downloaded: ${archive}"
    }
}
```

---

### Example 2: Custom Validation Task

```groovy
tasks.register('validateExtensions') {
    group = 'verification'
    description = 'Validate all extension DLLs'
    
    doLast {
        def extDir = file('ext')
        if (!extDir.exists()) {
            throw new GradleException('ext/ directory not found')
        }
        
        extDir.listFiles().each { dll ->
            if (dll.name.endsWith('.dll')) {
                println "Validating: ${dll.name}"
                validateDllArchitecture(dll)
            }
        }
        
        println '[SUCCESS] All extensions validated'
    }
}
```

---

### Example 3: Custom Info Task

```groovy
tasks.register('extensionInfo') {
    group = 'help'
    description = 'Display extension information'
    
    doLast {
        def extDir = file('ext')
        if (!extDir.exists()) {
            println 'No extensions directory found'
            return
        }
        
        println 'Installed Extensions:'
        println '-'.multiply(60)
        
        extDir.listFiles().each { dll ->
            if (dll.name.endsWith('.dll')) {
                def name = dll.name.replace('php_', '').replace('.dll', '')
                def size = dll.length() / 1024
                println "  ${name.padRight(20)} ${size} KB"
            }
        }
        
        println '-'.multiply(60)
    }
}
```

---

**Last Updated**: 2025-01-31  
**Version**: 2025.10.31
