# Gradle Build Comparison: module-php vs module-mysql

## Critical Issues Found in module-php build.gradle

### 1. **CRITICAL: Incorrect Archive Packaging Structure**

**Issue**: The PHP build.gradle packages archives incorrectly compared to MySQL reference.

**MySQL Pattern (CORRECT)**:
```groovy
// Archive contains: mysql{version}/ at root
// Example: mysql9.4.0/bin/mysqld.exe
from(mysqlPrepPath.parentFile) {
    include "${mysqlPrepPath.name}/**"
}
```

**PHP Current (INCORRECT)**:
```groovy
// Archive should contain: php{version}/ at root
// But current implementation is unclear/inconsistent
```

**Fix Required**: Update packaging to ensure `php{version}/` is at archive root, not `{bundle.release}/`

---

### 2. **Missing: modules-untouched Integration**

**Issue**: PHP build lacks integration with modules-untouched repository for version management.

**MySQL Has**:
- `fetchModulesUntouchedProperties()` function
- `downloadFromModulesUntouched()` function
- `checkModulesUntouched` task
- Automatic download of missing binaries

**PHP Missing**:
- No modules-untouched integration
- No automatic download capability
- Manual binary management only

**Impact**: Users must manually download PHP binaries, unlike MySQL which auto-downloads.

---

### 3. **Missing: findMysqlDirectory() Equivalent**

**Issue**: PHP build lacks intelligent directory detection after extraction.

**MySQL Has**:
```groovy
def findMysqlDirectory(File extractPath) {
    // Recursively searches for bin/mysqld.exe
    // Handles various archive structures
}
```

**PHP Missing**: No equivalent `findPhpDirectory()` function.

**Impact**: Cannot handle varied archive structures automatically.

---

### 4. **Missing: find7ZipExecutable() Function**

**Issue**: PHP build references `find7ZipExecutable()` but doesn't define it.

**Error Location**: Line ~1050 in releaseBuild task:
```groovy
def sevenZipPath = find7ZipExecutable()  // UNDEFINED!
```

**MySQL Has**: Complete implementation checking:
- Environment variable `7Z_HOME`
- Common installation paths
- PATH environment variable

**Fix Required**: Copy function from MySQL build.gradle.

---

### 5. **Incorrect: Output Directory Structure**

**Issue**: PHP uses different output path than MySQL.

**MySQL Pattern (CORRECT)**:
```groovy
// bearsampp-build/bins/mysql/{bundle.release}/bearsampp-mysql-{version}-{bundle.release}.7z
def buildBinsPath = file("${buildPath}/${bundleType}/${bundleName}/${bundleRelease}")
```

**PHP Current**:
```groovy
// Uses moduleBuildOutputPath which is defined as:
moduleBuildOutputPath = file("${buildBasePath}/${bundleType}/${bundleName}/${bundleRelease}").absolutePath
```

**Status**: Actually CORRECT, but variable naming is inconsistent with MySQL.

---

### 6. **Missing: releaseAll Task**

**Issue**: PHP build lacks batch processing capability.

**MySQL Has**: `releaseAll` task to build all versions in bin/ directory.

**PHP Missing**: No equivalent task.

**Impact**: Cannot build multiple versions in one command.

---

### 7. **Inconsistent: Version Resolution**

**Issue**: PHP has complex version resolution with multiple tasks.

**PHP Has**:
- `resolveVersion` task
- `releaseInteractive` task  
- `release` task
- Complex provider pattern

**MySQL Has**: Simpler inline resolution in `release` task.

**Issue**: PHP's approach is more complex but may have configuration cache issues.

---

### 8. **Missing: Archive Root Folder Validation**

**Issue**: PHP documentation claims archive contains `{bundle.release}/` at root, but code suggests `php{version}/`.

**Documentation Says**:
```
Content root: the top-level folder inside the archive is `{bundle.release}/` (e.g., `2025.10.31/`).
Inside `{bundle.release}/` the structure mirrors Bruno:
  - `bin/php{version}/...`
```

**Code Does**:
```groovy
// packageRelease7z and packageReleaseZip include bundleFolder at root
// bundleFolder = "${bundleName}${versionToBuild}" = "php8.3.15"
```

**Conflict**: Documentation and code don't match!

---

### 9. **Missing: Helper Functions**

**MySQL Has**:
- `calculateHash()` - defined as helper function
- `generateHashFiles()` - defined as helper function
- `find7ZipExecutable()` - defined as helper function
- `findMysqlDirectory()` - defined as helper function

**PHP Has**:
- `generateHashes` task (not a function)
- Missing `find7ZipExecutable()` entirely
- No directory finding logic

**Impact**: Code duplication and missing functionality.

---

### 10. **Incorrect: Hash Generation Task**

**Issue**: PHP's `generateHashes` task uses provider pattern but may fail.

**PHP Current**:
```groovy
tasks.register('generateHashes') {
    doLast {
        def versionToBuild = bundleVersionProvider.getOrNull()
        // ... uses bundleVersionProvider
    }
}
```

**MySQL Pattern**:
```groovy
// Hash generation is inline in release task
generateHashFiles(archiveFile)
```

**Issue**: Provider may not be set when task runs independently.

---

### 11. **Missing: Proper 7z Command Structure**

**Issue**: PHP's 7z packaging doesn't match MySQL's proven pattern.

**MySQL (CORRECT)**:
```groovy
def command = [
    sevenZipExe,
    'a',
    '-t7z',
    archiveFile.absolutePath.toString(),
    "${bundleName}${bundleVersion}"
]
def process = new ProcessBuilder(command as String[])
    .directory(file(bundleTmpPrepPath))
    .redirectErrorStream(true)
    .start()
```

**PHP Current**:
```groovy
executable '7z'
args 'a', '-t7z', archiveFile.absolutePath, bundleFolder
workingDir prepRoot
```

**Issue**: Uses Exec task instead of ProcessBuilder, less control over output.

---

### 12. **Missing: Proper Error Messages**

**MySQL Has**: Detailed error messages with suggestions:
```groovy
throw new GradleException("""
    Failed to download from modules-untouched: ${e.message}

    Tried URL: ${untouchedUrl}

    Please verify:
    1. Version ${version} exists in modules-untouched repository
    2. The URL is correct in mysql.properties
    3. You have internet connectivity
""".stripIndent())
```

**PHP Has**: Basic error messages without context.

---

## Summary of Required Fixes

### Priority 1 (Critical - Breaks Build)
1. ✅ Add `find7ZipExecutable()` function
2. ✅ Fix archive packaging structure (ensure php{version}/ at root)
3. ��� Update documentation to match actual behavior

### Priority 2 (Important - Missing Features)
4. ⚠️ Add modules-untouched integration (optional for PHP)
5. ⚠️ Add `releaseAll` task
6. ⚠️ Add `findPhpDirectory()` function for extraction

### Priority 3 (Nice to Have - Improvements)
7. 📝 Simplify version resolution (remove complex provider pattern)
8. 📝 Convert hash generation to helper function
9. 📝 Improve error messages
10. 📝 Add better extraction handling

---

## Recommended Actions

1. **Immediate**: Copy `find7ZipExecutable()` from MySQL build
2. **Immediate**: Fix archive packaging to match MySQL pattern
3. **Immediate**: Update documentation to reflect actual structure
4. **Soon**: Add `releaseAll` task for batch processing
5. **Consider**: Add modules-untouched integration if PHP binaries should auto-download
6. **Consider**: Simplify version resolution to match MySQL's simpler approach

---

## Files to Update

1. `build.gradle` - Add missing functions, fix packaging
2. `.gradle-docs/README.md` - Fix documentation about archive structure
3. `.gradle-docs/CONFIGURATION.md` - Update if exists
4. `.gradle-docs/API.md` - Document helper functions
