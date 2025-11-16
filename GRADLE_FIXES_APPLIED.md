# Gradle Build Fixes Applied to module-php

## Date: 2025-01-31

## Summary

Fixed critical errors in `build.gradle` by comparing against the reference MySQL module implementation from https://github.com/Bearsampp/module-mysql/blob/gradle-convert/build.gradle.

---

## Critical Fixes Applied

### 1. ✅ Added Missing `find7ZipExecutable()` Function

**Issue**: Function was referenced but not defined, causing build failures.

**Fix**: Added complete implementation from MySQL module:
- Checks `7Z_HOME` environment variable
- Checks common installation paths
- Searches PATH for 7z.exe
- Returns null if not found

**Location**: Lines ~320-355 in build.gradle

---

### 2. ✅ Added Missing Helper Functions

**Issue**: Hash generation and calculation functions were missing.

**Fix**: Added three helper functions from MySQL module:
- `calculateHash(File file, String algorithm)` - Calculates hash for a file
- `generateHashFiles(File file)` - Generates .md5, .sha1, .sha256, .sha512 files
- Integrated into packaging process

**Location**: Lines ~357-410 in build.gradle

---

### 3. ✅ Fixed 7z Packaging Implementation

**Issue**: Used Gradle Exec task instead of ProcessBuilder, less control over output.

**Fix**: Converted to ProcessBuilder pattern matching MySQL:
- Better error handling
- Proper output streaming
- Clearer error messages
- Integrated hash generation directly into packaging task

**Location**: Lines ~780-850 in build.gradle (packageRelease7z task)

---

### 4. ✅ Improved Error Messages

**Issue**: Generic error messages without context.

**Fix**: Added detailed error messages with:
- Download link for 7-Zip
- Instructions for setting 7Z_HOME
- Clear indication of what went wrong

---

## Documentation Issues Identified

### ⚠️ README.md Archive Structure Documentation

**Issue**: Documentation claims archive contains `{bundle.release}/` at root (e.g., `2025.10.31/`), but code actually creates `php{version}/` at root (e.g., `php8.3.15/`).

**Current Code Behavior**:
```groovy
// Archive contains: php8.3.15/ at root
def bundleFolder = "${bundleName}${versionToBuild}"  // = "php8.3.15"
// ProcessBuilder creates archive with bundleFolder at root
```

**Documentation Says**:
```
Content root: the top-level folder inside the archive is `{bundle.release}/` (e.g., `2025.10.31/`).
Inside `{bundle.release}/` the structure mirrors Bruno:
  - `bin/php{version}/...`
```

**Recommendation**: Update `.gradle-docs/README.md` line ~250-270 to match actual behavior:
```markdown
### Packaging Details

- Archive name format: `bearsampp-php-{version}-{bundle.release}.{7z|zip}`
- Location: `bearsampp-build/bins/php/{bundle.release}/`
- Content root: `php{version}/` at archive root (e.g., `php8.3.15/`)
- Matches MySQL module pattern: `mysql{version}/` at root

To verify:
```bash
7z l bearsampp-build/bins/php/2025.10.31/bearsampp-php-8.3.15-2025.10.31.7z
# Should show: php8.3.15/php.exe, php8.3.15/ext/, etc.
```
```

---

## Features Still Missing (Compared to MySQL)

### 1. ⚠️ modules-untouched Integration

**MySQL Has**: Automatic download of missing binaries from modules-untouched repository.

**PHP Missing**: No integration with modules-untouched for PHP binaries.

**Impact**: Users must manually download PHP binaries.

**Recommendation**: Consider adding if PHP binaries should auto-download like MySQL.

---

### 2. ⚠️ releaseAll Task

**MySQL Has**: `releaseAll` task to build all versions in bin/ directory.

**PHP Missing**: No batch processing capability.

**Impact**: Cannot build multiple versions in one command.

**Recommendation**: Add `releaseAll` task for convenience:
```groovy
tasks.register('releaseAll') {
    group = 'build'
    description = 'Build release packages for all available versions'
    // Implementation similar to MySQL module
}
```

---

### 3. ⚠️ findPhpDirectory() Function

**MySQL Has**: `findMysqlDirectory()` to intelligently locate binaries after extraction.

**PHP Missing**: No equivalent function.

**Impact**: Cannot handle varied archive structures automatically.

**Recommendation**: Add if PHP binaries come in varied archive structures.

---

## Files Modified

1. **build.gradle** - Added missing functions, fixed packaging
2. **GRADLE_COMPARISON.md** - Created detailed comparison document
3. **GRADLE_FIXES_APPLIED.md** - This file

---

## Files That Need Manual Update

1. **.gradle-docs/README.md** - Fix archive structure documentation (lines ~250-270)
2. **.gradle-docs/API.md** - Document new helper functions (if exists)
3. **.gradle-docs/TASKS.md** - Update task descriptions (if exists)

---

## Testing Recommendations

### 1. Test 7z Packaging
```bash
gradle clean
gradle release -PbundleVersion=8.3.15
# Verify archive created in bearsampp-build/bins/php/2025.10.31/
# Verify hash files (.md5, .sha1, .sha256, .sha512) created
```

### 2. Test Zip Packaging
```bash
# Change bundle.format=zip in build.properties
gradle clean
gradle release -PbundleVersion=8.3.15
# Verify .zip archive created
```

### 3. Test Archive Structure
```bash
7z l bearsampp-build/bins/php/2025.10.31/bearsampp-php-8.3.15-2025.10.31.7z
# Should show php8.3.15/ at root, not 2025.10.31/
```

### 4. Test Without 7-Zip
```bash
# Temporarily rename 7-Zip directory
gradle release -PbundleVersion=8.3.15
# Should show clear error message with download link
```

---

## Comparison with MySQL Module

| Feature | MySQL | PHP (Before) | PHP (After) | Status |
|---------|-------|--------------|-------------|--------|
| find7ZipExecutable() | ✅ | ❌ | ✅ | Fixed |
| calculateHash() | ✅ | ❌ | ✅ | Fixed |
| generateHashFiles() | ✅ | ❌ | ✅ | Fixed |
| ProcessBuilder packaging | ✅ | ❌ | ✅ | Fixed |
| Hash generation | ✅ | ⚠️ | ✅ | Fixed |
| modules-untouched | ✅ | ❌ | ❌ | Not needed? |
| releaseAll task | ✅ | ❌ | ❌ | Optional |
| findDirectory() | ✅ | ❌ | ❌ | Optional |
| Archive structure | mysql{ver}/ | php{ver}/ | php{ver}/ | Correct |
| Documentation | ✅ | ❌ | ⚠️ | Needs update |

---

## Next Steps

1. **Immediate**: Test the build with a real PHP version
2. **Soon**: Update README.md documentation to match actual behavior
3. **Consider**: Add `releaseAll` task for batch processing
4. **Consider**: Add modules-untouched integration if needed
5. **Optional**: Add `findPhpDirectory()` if archive structures vary

---

## Conclusion

The critical build errors have been fixed by adding missing functions from the MySQL reference implementation. The build should now work correctly for packaging PHP modules.

The main remaining issue is documentation that doesn't match the actual behavior - the archives contain `php{version}/` at root, not `{bundle.release}/`. This is actually correct and matches the MySQL pattern, but the documentation needs updating.

---

**Fixed By**: Qodo AI Assistant  
**Date**: 2025-01-31  
**Reference**: https://github.com/Bearsampp/module-mysql/blob/gradle-convert/build.gradle
