# PHP Extension Testing CI/CD Workflow

## Overview

This document describes the automated PHP extension validation workflow that runs on every pull request to the `main` branch. The workflow is designed to ensure that new PHP versions and their extensions are properly configured and functional across multiple Windows environments.

## Workflow Location

**File:** `.github/workflows/php-extension-test.yml`

**Trigger:** Automatically runs when a PR is opened, synchronized, or reopened against the `main` branch.

---

## Current Status

### ✅ What is Currently Working

The workflow infrastructure is fully operational and includes:

1. **Automated PR Detection**
   - Triggers on PR creation, updates, and reopening
   - Monitors changes to `releases.properties` file

2. **Version Detection**
   - Automatically detects new PHP versions added to `releases.properties`
   - Extracts version numbers from git diff
   - Parses version entries for testing

3. **Multi-Platform Testing Matrix**
   - Tests across 4 Windows configurations:
     - Windows 10 AMD
     - Windows 10 Intel
     - Windows 11 AMD
     - Windows 11 Intel
   - Parallel execution with `fail-fast: false` to test all platforms

4. **Result Aggregation**
   - Collects results from all platform tests
   - Generates visual status badges for each platform
   - Posts consolidated results as a single PR comment

5. **Artifact Management**
   - Uploads test results as artifacts
   - Preserves results for debugging and analysis

### ⚠️ What is NOT Yet Implemented

The core validation logic is **missing**. The workflow contains a placeholder:

```powershell
# TODO: Insert your extension validation logic here
```

This means the workflow runs successfully but **does not actually validate PHP extensions**.

---

## What Needs to Be Tested

Based on the repository structure (`bin/php{version}/` directories), each PHP version includes:

### 1. PHP Extensions (`exts.properties`)

Each PHP version defines external extensions that must be validated:

- **imagick** - ImageMagick image processing extension
- **memcache** - Memcache caching extension
- **xdebug** - Xdebug debugging extension

### 2. Dependencies (`deps.properties`)

External dependencies required by extensions:

- **ImageMagick** - Binary dependency for imagick extension

### 3. Configuration Files

- `php.ini` - PHP configuration with extension paths
- `bearsampp.conf` - Bearsampp-specific configuration
- `pear.properties` - PEAR package manager configuration

---

## Required Implementation Tasks

### Phase 1: Basic PHP Validation

#### Task 1.1: Download and Extract PHP
```powershell
# Parse releases.properties to get download URL
# Download the PHP 7z archive
# Extract to temporary directory
# Verify php.exe exists and is executable
```

**Validation Points:**
- Archive downloads successfully
- Extraction completes without errors
- PHP executable is present
- PHP version matches expected version

#### Task 1.2: Verify PHP Executable
```powershell
# Run: php -v
# Verify version output matches expected
# Check for any startup errors or warnings
```

**Validation Points:**
- PHP starts without errors
- Version string is correct
- No missing DLL errors

---

### Phase 2: Extension Validation

#### Task 2.1: Download Extensions
```powershell
# Parse bin/php{version}/exts.properties
# Download each extension from specified URL
# Verify download integrity
# Extract if archived (e.g., imagick comes as .zip)
```

**Validation Points:**
- All extension URLs are accessible
- Downloads complete successfully
- Archives extract properly
- DLL files are present

#### Task 2.2: Validate DLL Architecture
```powershell
# Use PowerShell to check DLL architecture
# Example:
$dllPath = "path/to/php_imagick.dll"
$peHeader = [System.Reflection.PortableExecutable.PEHeaders]::new(
    [System.IO.FileStream]::new($dllPath, [System.IO.FileMode]::Open)
)
$architecture = $peHeader.CoffHeader.Machine
# Verify: AMD64 for x64, I386 for x86
```

**Validation Points:**
- Extension DLL architecture matches PHP architecture (x64)
- Thread-safe (TS) vs non-thread-safe (NTS) compatibility
- Visual Studio version compatibility (VS16, VS17, etc.)

#### Task 2.3: Test Extension Loading
```powershell
# For each extension, test loading:
php -d extension=php_imagick.dll -m | Select-String "imagick"
php -d extension=php_memcache.dll -m | Select-String "memcache"
php -d zend_extension=php_xdebug.dll -m | Select-String "xdebug"
```

**Validation Points:**
- Extension appears in `php -m` output
- No error messages during loading
- No missing dependency errors

---

### Phase 3: Dependency Validation

#### Task 3.1: Download Dependencies
```powershell
# Parse bin/php{version}/deps.properties
# Download ImageMagick portable package
# Extract to appropriate location
```

**Validation Points:**
- Dependency downloads successfully
- Extraction completes
- Required DLLs are present

#### Task 3.2: Test Extension with Dependencies
```powershell
# Set PATH to include dependency DLLs
$env:PATH = "path/to/imagemagick;$env:PATH"
# Test imagick extension loads with ImageMagick present
php -d extension=php_imagick.dll -r "echo class_exists('Imagick') ? 'OK' : 'FAIL';"
```

**Validation Points:**
- Extension loads with dependencies
- Extension classes/functions are available
- No runtime errors

---

### Phase 4: Functional Testing (Optional)

#### Task 4.1: Basic Functionality Tests
```powershell
# Test imagick can create an image
php -d extension=php_imagick.dll -r "
  $img = new Imagick();
  $img->newImage(100, 100, new ImagickPixel('red'));
  echo 'imagick: OK';
"

# Test memcache extension functions exist
php -d extension=php_memcache.dll -r "
  echo function_exists('memcache_connect') ? 'memcache: OK' : 'FAIL';
"

# Test xdebug is loaded
php -d zend_extension=php_xdebug.dll -r "
  echo extension_loaded('xdebug') ? 'xdebug: OK' : 'FAIL';
"
```

**Validation Points:**
- Extensions provide expected functionality
- No runtime errors during basic operations
- Classes and functions are accessible

---

### Phase 5: Error Handling and Reporting

#### Task 5.1: Capture Detailed Errors
```powershell
# Capture stderr and stdout
# Log all PHP warnings and errors
# Save failed DLL information
```

#### Task 5.2: Enhanced PR Comments
```powershell
# Include in PR comment:
# - PHP version tested
# - Extension test results (pass/fail per extension)
# - Specific error messages
# - Download URLs that failed
# - Architecture mismatches
# - Missing dependencies
```

**Example Enhanced Comment:**
```markdown
### ✅ PHP Extension Tests - Partial Success

#### PHP 8.3.27
- ✅ win10-amd: All tests passed
- ❌ win10-intel: imagick failed to load
  - Error: Missing CORE_RL_MagickCore_.dll
  - ImageMagick dependency not found
- ✅ win11-amd: All tests passed
- ✅ win11-intel: All tests passed

#### PHP 8.4.14
- ✅ All platforms: All tests passed

**Extensions Tested:**
- ✅ imagick (3.7.0)
- ✅ memcache (8.3.x)
- ✅ xdebug (3.5.0alpha2)
```

---

## Implementation Priority

### High Priority (Must Have)
1. ✅ Download and extract PHP version
2. ✅ Verify PHP executable runs
3. ✅ Download extensions from exts.properties
4. ✅ Validate DLL architecture
5. ✅ Test extension loading with `php -m`

### Medium Priority (Should Have)
6. ✅ Download and validate dependencies
7. ✅ Test extensions with dependencies present
8. ✅ Enhanced error reporting in PR comments
9. ✅ Upload failed artifacts for debugging

### Low Priority (Nice to Have)
10. ⭕ Basic functionality tests per extension
11. ⭕ Performance benchmarks
12. ⭕ Memory leak detection
13. ⭕ Compatibility matrix documentation

---

## Testing Workflow Locally

To test the validation logic locally before committing:

```powershell
# 1. Navigate to module-php directory
cd E:\Bearsampp-development\module-php

# 2. Get a PHP version from releases.properties
$version = "8.3.27"
$url = (Get-Content releases.properties | Select-String "^$version =").Line.Split('=')[1].Trim()

# 3. Download and extract
Invoke-WebRequest -Uri $url -OutFile "php-$version.7z"
7z x "php-$version.7z" -o"test-php-$version"

# 4. Test PHP
& "test-php-$version\php.exe" -v

# 5. Download and test extensions
$exts = Get-Content "bin\php$version\exts.properties"
# ... continue with extension testing
```

---

## Architecture Validation Script

Here's a PowerShell script to validate DLL architecture (to be integrated into workflow):

```powershell
function Test-DllArchitecture {
    param(
        [string]$DllPath,
        [string]$ExpectedArch = "AMD64"  # or "I386" for x86
    )
    
    try {
        $fs = [System.IO.FileStream]::new($DllPath, [System.IO.FileMode]::Open, [System.IO.FileAccess]::Read)
        $br = [System.IO.BinaryReader]::new($fs)
        
        # Read DOS header
        $dosHeader = $br.ReadBytes(64)
        $peOffset = [BitConverter]::ToInt32($dosHeader, 60)
        
        # Seek to PE header
        $fs.Seek($peOffset, [System.IO.SeekOrigin]::Begin) | Out-Null
        $peSignature = $br.ReadBytes(4)
        
        # Read COFF header
        $machine = $br.ReadUInt16()
        
        $fs.Close()
        
        $arch = switch ($machine) {
            0x014c { "I386" }   # x86
            0x8664 { "AMD64" }  # x64
            default { "UNKNOWN" }
        }
        
        return @{
            Success = ($arch -eq $ExpectedArch)
            Detected = $arch
            Expected = $ExpectedArch
        }
    }
    catch {
        return @{
            Success = $false
            Error = $_.Exception.Message
        }
    }
}

# Usage:
$result = Test-DllArchitecture -DllPath "php_imagick.dll" -ExpectedArch "AMD64"
if ($result.Success) {
    Write-Host "✅ DLL architecture is correct: $($result.Detected)"
} else {
    Write-Host "❌ DLL architecture mismatch: Expected $($result.Expected), got $($result.Detected)"
}
```

---

## File Structure Reference

Each PHP version in the repository follows this structure:

```
bin/
└── php{version}/
    ├── bearsampp.conf      # Bearsampp configuration
    ├── deps.properties     # External dependencies (ImageMagick, etc.)
    ├── exts.properties     # PHP extensions (imagick, memcache, xdebug)
    ���── pear.properties     # PEAR configuration
    ├── php.ini             # PHP configuration
    └── deps/
        └── README.txt      # Dependency documentation
```

**Example for PHP 8.3.27:**
- `bin/php8.3.27/exts.properties` - Lists 3 extensions
- `bin/php8.3.27/deps.properties` - Lists ImageMagick dependency
- `bin/php8.3.27/php.ini` - Contains extension configuration

---

## Related Documentation

- **Workflow File:** `.github/workflows/php-extension-test.yml`
- **Build System:** [.gradle-docs/README.md](../../.gradle-docs/README.md)
- **Task Reference:** [.gradle-docs/TASKS.md](../../.gradle-docs/TASKS.md)
- **Configuration Guide:** [.gradle-docs/CONFIGURATION.md](../../.gradle-docs/CONFIGURATION.md)

---

## Contributing

To contribute to the extension testing workflow:

1. Review this document to understand current status
2. Implement one or more tasks from the priority list
3. Test locally using the provided scripts
4. Submit a PR with your changes
5. The workflow will run on your PR (though validation is incomplete)

---

## Questions or Issues?

- **Workflow Issues:** Check `.github/workflows/php-extension-test.yml`
- **Extension Issues:** Check `bin/php{version}/exts.properties`
- **Dependency Issues:** Check `bin/php{version}/deps.properties`
- **General Issues:** [Report on Bearsampp repository](https://github.com/bearsampp/bearsampp/issues)

---

**Last Updated:** 2025-02-18  
**Status:** 🚧 Workflow infrastructure complete, validation logic pending implementation
