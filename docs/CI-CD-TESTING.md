# PHP Module CI/CD Testing

## Overview

This document describes the automated testing workflow for the Bearsampp PHP module. The CI/CD pipeline ensures that PHP builds are properly packaged, can be extracted, that all executables are functional, and that PHP extensions load correctly with their dependencies.

## Workflow Triggers

The PHP testing workflow is triggered automatically on:

- **Pull Request Events**: When a PR is opened, synchronized (new commits pushed), reopened, or edited
  - Target branches: `main`
  - Monitored files: `bin/**`
  - Note: `releases.properties` is not monitored as it's always updated before PR creation
- **Manual Workflow Dispatch**: Can be triggered manually with options to test specific versions and platforms

## Test Scope

The workflow intelligently determines which versions to test based on the context using a three-tier detection system:

### Pull Request Testing

**Smart Detection with Three-Tier Fallback System:**

1. **Primary Method: /bin Directory Detection** (Most Accurate)
   - Automatically detects which PHP versions are included in the PR from changed files in `/bin` directory
   - Extracts version numbers from directory names (e.g., `bin/php8.3.27/` → `8.3.27`)
   - Verifies detected versions exist in `releases.properties`
   - **Use Case**: New versions are created and added to a pre-release (tagged with date, e.g., "2025.11.23")
   - **How it works**:
     - Version directories are created in `/bin` (e.g., `bin/php8.3.27/`, `bin/php8.4.0/`)
     - The `releases.properties` file is updated via the releases.properties workflow
     - A PR is created from a release branch (e.g., "November") to main
     - This workflow detects changed files in `/bin` and extracts version numbers from directory names
     - Tests are run against the version(s) found in `releases.properties` from that PR branch

2. **Fallback Method: PR Title Detection**
   - If no versions found in `/bin`, extracts version numbers from PR title
   - Supports patterns like "8.3.27", "8.4.0" in PR titles
   - Verifies these versions exist in `releases.properties`
   - **Example**: PR title "Update docs for PHP 8.3.27" → tests version 8.3.27

3. **Final Fallback: Latest 5 Versions**
   - If no versions detected by either method, tests the latest 5 versions from `releases.properties`
   - Ensures critical versions are always tested even when detection fails
   - Versions are sorted and the 5 most recent are selected

**Benefits:**
- **Efficiency**: Reduces CI runtime by testing only relevant versions
- **Reliability**: Multiple fallback methods ensure tests always run when needed
- **Flexibility**: Adapts to different PR workflows and scenarios
- **Multi-Platform**: Tests across Windows 10 and 11 with AMD and Intel configurations

### Manual Testing
- **Specific Version**: Tests a single version provided as input parameter
- **Platform Selection**: Choose specific platform or test all platforms
- **Flexibility**: Useful for re-testing or validating specific versions

### Example Scenarios
- **Add PHP 8.3.27 via /bin**: Version directory `bin/php8.3.27/` is created → Only version 8.3.27 is tested
- **Multiple versions in PR**: `bin/php8.3.27/` and `bin/php8.4.0/` are added → Both versions are tested
- **PR title "Update docs for PHP 8.4.0"**: Version 8.4.0 is tested if it exists in `releases.properties` (fallback method)
- **Documentation-only PR**: No versions detected → Latest 5 versions are tested (final fallback)
- **Modify existing version URL**: That specific version is tested

## Test Phases

### Phase 1: Basic PHP Validation

**Purpose**: Verify the PHP package can be downloaded, extracted, and executed correctly.

**Steps**:
1. **Download PHP** (Phase 1.1)
   - Reads the download URL from `releases.properties`
   - Downloads the `.7z` archive from the specified source
   - Reports download size and success status
   - Validates file size (must be > 0.1 MB)
   - Includes timeout protection (300 seconds)

2. **Extract Archive** (Phase 1.1)
   - Extracts the `.7z` file using 7-Zip
   - Locates the `php.exe` executable (may be nested)
   - Validates the directory structure
   - Reports extraction success with file location

3. **Verify PHP Executable** (Phase 1.2)
   - Tests `php.exe -v` to verify version information
   - Validates version string matches expected version
   - Tests PHP code execution with simple echo command
   - Confirms PHP is fully functional

### Phase 2: Extension Validation

**Purpose**: Verify PHP extensions can be downloaded, have correct architecture, and load properly.

**Steps**:
1. **Download Extensions** (Phase 2.1)
   - Reads extension URLs from `bin/php{version}/exts.properties`
   - Downloads each extension package
   - Extracts ZIP archives if needed
   - Reports download size and success for each extension
   - Saves results to JSON for artifact analysis

2. **Validate DLL Architecture** (Phase 2.2)
   - Scans all downloaded DLL files
   - Reads PE header to determine architecture (x86 vs x64)
   - Validates all DLLs are AMD64 (x64) architecture
   - Reports any architecture mismatches
   - Saves validation results to JSON

3. **Test Extension Loading** (Phase 2.3)
   - Attempts to load each extension with PHP
   - Uses `extension=` directive for regular extensions
   - Uses `zend_extension=` directive for Zend extensions (e.g., xdebug)
   - Verifies extension appears in `php -m` output
   - Reports loading success/failure for each extension

### Phase 3: Dependency Validation

**Purpose**: Verify extensions work correctly with their required dependencies.

**Steps**:
1. **Download Dependencies** (Phase 3.1)
   - Reads dependency URLs from `bin/php{version}/deps.properties`
   - Downloads packages like ImageMagick, memcache servers, etc.
   - Extracts archives (supports .7z and .zip)
   - Reports download and extraction status
   - Saves results to JSON

2. **Test Extensions with Dependencies** (Phase 3.2)
   - Adds dependency directories to system PATH
   - Tests imagick extension with ImageMagick
     - Verifies `Imagick` class exists
   - Tests memcache extension
     - Verifies `memcache_connect` function exists
   - Tests xdebug extension
     - Verifies extension loads as Zend extension
   - Reports success/failure for each dependency test

### Phase 4: Functional Testing

**Purpose**: Verify extensions can perform their intended functions.

**Steps**:
1. **Test Imagick Functionality**
   - Creates a new image object
   - Generates a 100x100 red PNG image
   - Verifies image creation succeeds
   - Tests core imagick functionality

2. **Test Memcache Functionality**
   - Verifies memcache functions are available
   - Note: Full testing requires running memcache server
   - Confirms function existence as baseline test

3. **Test Xdebug Functionality**
   - Verifies xdebug extension is loaded
   - Confirms Zend extension loading works
   - Tests basic xdebug availability

## Test Results

### Artifacts

The workflow generates and uploads the following artifacts for each version/platform combination:

1. **Test Results** (`test-results-VERSION-PLATFORM`)
   - `extensions.json` - Extension download results
   - `architecture.json` - DLL architecture validation results
   - `loading.json` - Extension loading test results
   - `dependencies.json` - Dependency download results
   - `dependency-tests.json` - Extension+dependency test results
   - `functional.json` - Functional test results
   - `summary-PLATFORM.md` - Human-readable test summary
   - Retention: 30 days

### PR Comments

For pull requests, the workflow automatically posts a comment with:
- Test date and time (UTC)
- Overall test status with emoji indicators
- Badge matrix showing pass/fail for each version/platform combination
- Test completion statistics
- Detailed results (only shown if there are failures)
- Test phase descriptions
- Links to detailed artifacts

The comment is updated if it already exists (no duplicate comments).

### Badge System

Each version/platform combination gets a badge:
- ![PASS](https://img.shields.io/badge/platform-PASS-success?style=flat-square&logo=php&logoColor=white) - All tests passed
- ![FAIL](https://img.shields.io/badge/platform-FAIL-critical?style=flat-square&logo=php&logoColor=white) - Some tests failed
- ![NO_RESULTS](https://img.shields.io/badge/platform-NO_RESULTS-inactive?style=flat-square&logo=php&logoColor=white) - No results available

### Test Summary Format

Each version/platform summary includes:

```
### PHP X.Y.Z - PLATFORM

**Phase 1: Basic PHP Validation**
- Download & Extract: ✅ PASS / ❌ FAIL
  - Error: [error message if failed]
- Verify Executable: ✅ PASS / ❌ FAIL
  - Error: [error message if failed]

**Phase 2: Extension Validation**
- Download Extensions: ✅ PASS / ❌ FAIL
- Validate Architecture: ✅ PASS / ❌ FAIL
- Test Loading: ✅ PASS / ❌ FAIL

**Phase 3: Dependency Validation**
- Download Dependencies: ✅ PASS / ❌ FAIL
- Test with Dependencies: ✅ PASS / ❌ FAIL

**Phase 4: Functional Testing**
- Extension Functionality: ✅ PASS / ⚠️ PARTIAL

**Extensions Tested:** N
- ✅ extension1
- ❌ extension2
- ✅ extension3

**Overall Status:** ✅ ALL TESTS PASSED / ❌ SOME TESTS FAILED
```

If tests fail, additional troubleshooting information is provided in a collapsible section.

## Error Handling

The workflow provides comprehensive error reporting at multiple levels:

### Detailed Error Messages

Each phase captures and reports specific error information:

- **Download failures**: 
  - HTTP status codes and error messages
  - Attempted URLs
  - File size validation errors
  - Timeout errors
  
- **Extraction failures**: 
  - 7-Zip exit codes and output logs
  - Directory listings showing actual structure
  - Expected vs. actual file patterns
  
- **PHP execution failures**: 
  - Command output and error messages
  - Exit codes
  - Version information (if available)
  
- **Extension loading failures**:
  - Missing DLL errors
  - Architecture mismatches
  - Dependency missing errors
  - PHP error output

### Error Propagation

- Each phase uses `continue-on-error: true` to allow subsequent phases to run
- Failed phases are clearly marked in the summary with ❌ indicators
- Error messages are included inline in test summaries
- All test results are uploaded regardless of success/failure
- Overall workflow status reflects any failures

### Troubleshooting Assistance

When tests fail, the summary includes:
- Specific error messages for each failed phase
- Collapsible troubleshooting tips section with:
  - Links to detailed workflow logs
  - Instructions to download test artifacts
  - Common issues and solutions
  - Archive structure validation tips
  - DLL dependency checks
  - Extension compatibility notes

## Platform Matrix

The workflow tests across multiple Windows configurations:

- **win10-amd**: Windows 10 with AMD processor configuration
- **win10-intel**: Windows 10 with Intel processor configuration
- **win11-amd**: Windows 11 with AMD processor configuration
- **win11-intel**: Windows 11 with Intel processor configuration

**Runner**: `windows-latest` (GitHub Actions Windows Server)
**Tools**: Native Windows PowerShell, 7-Zip (pre-installed)

## Matrix Strategy

- **Parallel Execution**: Tests run in parallel for different version/platform combinations
- **Fail-Fast**: Disabled (`fail-fast: false`) to test all combinations even if one fails
- **Dynamic Matrix**: Version and platform lists are generated dynamically based on trigger context
- **Resource Efficiency**: Only tests relevant versions to minimize CI time

## Workflow File Location

`.github/workflows/php-extension-test.yml`

## Maintenance

### Adding New Test Cases

To add new functionality tests:

1. Add a new step after the appropriate phase
2. Use `continue-on-error: true` to prevent blocking other tests
3. Check previous phase success with `if: steps.PREVIOUS_STEP.outputs.success == 'true'`
4. Set output variable: `echo "success=true/false" >> $env:GITHUB_OUTPUT`
5. Update the test summary generation to include the new phase
6. Document the new phase in this file

### Adding New Extensions to Test

To test additional PHP extensions:

1. Add the extension URL to `bin/php{version}/exts.properties`:
   ```
   extension_name = https://url/to/extension.zip
   ```
2. If the extension requires dependencies, add them to `bin/php{version}/deps.properties`
3. The workflow will automatically detect and test the new extension
4. Consider adding specific functional tests for the extension

### Modifying Platform Selection

To change which platforms are tested, edit the `Determine platforms to test` step:

```bash
# Current: All 4 platforms
echo "platforms=[\"win10-amd\",\"win10-intel\",\"win11-amd\",\"win11-intel\"]" >> $GITHUB_OUTPUT

# Example: Only Windows 11
echo "platforms=[\"win11-amd\",\"win11-intel\"]" >> $GITHUB_OUTPUT
```

### Adjusting Test Parameters

Key parameters that can be adjusted:

- **Download timeout**: `TimeoutSec 300` (5 minutes) in Phase 1.1
- **Minimum file size**: `0.1 MB` validation in Phase 1.1
- **Artifact retention**: `retention-days: 30` in Upload Test Results step

## Troubleshooting

### Common Issues

1. **Download Failures**
   - **Symptom**: Phase 1.1 fails with download error
   - **Causes**:
     - Invalid URL in `releases.properties`
     - Source server unavailable
     - Network connectivity issues
     - Rate limiting
   - **Solutions**:
     - Verify the URL is accessible in a browser
     - Check source server status
     - Ensure the version number matches exactly
     - Wait and retry if rate limited

2. **Extraction Failures**
   - **Symptom**: Phase 1.1 fails after successful download
   - **Causes**:
     - Corrupted `.7z` file
     - Incorrect archive structure
     - Insufficient disk space
   - **Solutions**:
     - Re-download and re-upload the archive
     - Verify archive structure locally before uploading
     - Check that archive contains `php.exe`

3. **PHP Execution Failures**
   - **Symptom**: Phase 1.2 fails when running PHP
   - **Causes**:
     - Missing DLL dependencies (MSVC runtime)
     - Corrupted PHP executable
     - Incompatible Windows version
   - **Solutions**:
     - Include all required DLLs in the archive
     - Test the build locally on Windows
     - Check for Visual C++ runtime dependencies

4. **Extension Loading Failures**
   - **Symptom**: Phase 2.3 fails to load extensions
   - **Causes**:
     - Architecture mismatch (x86 vs x64)
     - Missing extension dependencies
     - Incompatible PHP version
     - Corrupted DLL files
   - **Solutions**:
     - Verify all DLLs are x64 architecture
     - Include all required dependency DLLs
     - Check extension compatibility with PHP version
     - Re-download extension packages

5. **Dependency Test Failures**
   - **Symptom**: Phase 3.2 fails with dependency errors
   - **Causes**:
     - Missing dependency files
     - Incorrect PATH configuration
     - Version incompatibility
   - **Solutions**:
     - Verify all dependency files are included
     - Check dependency versions match extension requirements
     - Test locally with same dependency versions

### Debugging Steps

To debug a specific version/platform failure:

1. **Check Workflow Logs**
   - Navigate to the failed workflow run in GitHub Actions
   - Expand the failed step to see detailed output
   - Look for specific error messages with ❌ indicators

2. **Download Artifacts**
   - Download the `test-results-VERSION-PLATFORM` artifact
   - Review JSON files for detailed test data
   - Check `summary-PLATFORM.md` for formatted results

3. **Local Testing**
   - Download the same `.7z` file from the source
   - Extract it locally on Windows
   - Try running PHP and loading extensions manually
   - Check for missing DLL errors

4. **Compare with Working Versions**
   - Compare directory structure with a passing version
   - Check file sizes and contents
   - Verify all dependencies are present
   - Compare extension versions

### Manual Workflow Testing

To manually test a specific version:

1. Go to the Actions tab in GitHub
2. Select "PHP Extension Testing" workflow
3. Click "Run workflow"
4. Enter the PHP version (e.g., "8.3.27")
5. Select platform (or "all" for all platforms)
6. Click "Run workflow"
7. Monitor the test results

## Best Practices

1. **Always test before merging**: The workflow runs on PRs to catch issues early
2. **Review test summaries**: Check PR comments for test results before merging
3. **Investigate failures**: Don't merge if tests fail without understanding why
4. **Keep versions updated**: Regularly update `releases.properties` with new PHP versions
5. **Monitor artifact storage**: Old artifacts are automatically cleaned up after retention period
6. **Test locally first**: Before creating a PR, test the PHP build locally on Windows
7. **Document changes**: Update this documentation when modifying the workflow
8. **Version consistency**: Ensure version numbers in filenames match `releases.properties`
9. **Extension compatibility**: Verify extensions are compatible with PHP version before adding
10. **Dependency versions**: Keep dependency versions up to date and compatible

## Version Naming Conventions

PHP versions in `releases.properties` should follow these patterns:

- **Major.Minor.Patch**: `8.3.27`, `8.4.0`, `8.2.15`
- **Major.Minor**: `8.3`, `8.4`
- **Legacy versions**: `7.4.33`, `5.6.40`

The workflow supports all version formats and will test them accordingly.

## Workflow Outputs

### Success Indicators

When all tests pass, you'll see:
- ✅ Green checkmark on the PR
- "All tests passed" status in PR comment
- All badges showing PASS status
- All phases marked with ✅ PASS
- Artifacts uploaded successfully
- Message: "_All tests passed successfully! ✨_"

### Failure Indicators

When tests fail, you'll see:
- ❌ Red X on the PR
- "Some tests failed" status in PR comment
- Failed badges showing FAIL status
- Failed phases marked with ❌ FAIL
- Error messages in the summary
- Troubleshooting tips displayed
- Detailed test results expanded

### Skipped Tests

When tests are skipped, you'll see:
- ⏭️ "Tests skipped" status
- Explanation of why tests were skipped
- Instructions on how to trigger tests
- No artifacts generated

## Related Documentation

- [PHP Official Documentation](https://www.php.org/docs.php)
- [PHP Extension Development](https://www.php.net/manual/en/internals2.php)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Bearsampp Project](https://github.com/Bearsampp)
- [Module PHP Repository](https://github.com/Bearsampp/module-php)
- [Manual Testing Guide](MANUAL-TESTING.md)
- [PHP Extension Testing Guide](PHP-EXTENSION-TESTING.md)

## Changelog

### Version History

- **Phase 1**: Basic PHP Validation (Download, Extract, Verify)
- **Phase 2**: Extension Validation (Download, Architecture, Loading)
- **Phase 3**: Dependency Validation (Download Dependencies, Test with Dependencies)
- **Phase 4**: Functional Testing (Test Extension Functionality)
- **Phase 5**: Enhanced Error Reporting (Detailed errors, badges, smart detection)

### Recent Improvements

- **Three-Tier Smart Detection System**: Primary detection from `/bin` directory changes, fallback to PR title, final fallback to latest 5 versions
- **Bin Directory Monitoring**: Automatically detects versions from changed files in `/bin/php{version}/` directories
- **Pre-release Workflow Support**: Optimized for pre-release → PR → main workflow with automatic version detection
- **Latest 5 Versions Fallback**: Ensures tests always run even when specific versions aren't detected
- **PR Title Version Detection**: Extracts version numbers from PR titles as fallback method
- **Badge System**: Visual status indicators for each version/platform combination
- **Enhanced Error Messages**: Detailed error reporting with emoji indicators
- **Collapsible Troubleshooting Tips**: Context-sensitive help for failed tests
- **Platform-Specific Testing**: Tests across Windows 10/11 with AMD/Intel configurations
- **Smart Test Skipping**: Only runs tests when relevant changes are detected

### Future Enhancements

Potential improvements for future versions:

1. **Performance Testing**: Add basic performance benchmarks for PHP execution
2. **Security Checks**: Verify default security settings and configurations
3. **Configuration Testing**: Test various php.ini configurations
4. **Composer Testing**: Verify Composer works with the PHP build
5. **Web Server Testing**: Test PHP with built-in web server
6. **Database Extension Testing**: Test MySQL, PostgreSQL, SQLite extensions
7. **Code Coverage**: Add code coverage for extension tests

## Contributing to Tests

To contribute improvements to the testing workflow:

1. Fork the repository
2. Create a feature branch
3. Modify `.github/workflows/php-extension-test.yml`
4. Update this documentation
5. Test your changes with manual workflow dispatch
6. Submit a pull request with:
   - Description of changes
   - Rationale for improvements
   - Test results showing the changes work
   - Updated documentation

## Support and Contact

For questions or issues with the CI/CD workflow:

- **GitHub Issues**: [Open an issue](https://github.com/Bearsampp/module-php/issues)
- **Discussions**: Use GitHub Discussions for questions
- **Pull Requests**: Submit improvements via PR

When reporting issues, please include:
- PHP version being tested
- Platform (win10-amd, win10-intel, win11-amd, win11-intel)
- Link to failed workflow run
- Error messages from logs
- Steps to reproduce (if applicable)
- Relevant artifact files

## Appendix

### Extension Properties File Format

`bin/php{version}/exts.properties`:
```properties
# Extension name = Download URL
imagick = https://example.com/php_imagick-3.7.0-8.3-ts-vs16-x64.zip
xdebug = https://example.com/php_xdebug-3.3.0-8.3-ts-vs16-x64.dll
memcache = https://example.com/php_memcache-8.2-8.3-ts-vs16-x64.zip
```

### Dependencies Properties File Format

`bin/php{version}/deps.properties`:
```properties
# Dependency name = Download URL
imagemagick = https://example.com/ImageMagick-7.1.1-portable-Q16-x64.zip
memcached = https://example.com/memcached-win64-1.6.17.zip
```

### Test Result JSON Schema

**extensions.json**:
```json
{
  "extension_name": {
    "success": true,
    "path": "path/to/extension"
  }
}
```

**architecture.json**:
```json
{
  "php_extension.dll": {
    "valid": true,
    "arch": "AMD64"
  }
}
```

**loading.json**:
```json
{
  "extension_name": {
    "loaded": true,
    "type": "extension"
  }
}
```
