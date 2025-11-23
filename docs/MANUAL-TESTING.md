# Manual Testing Guide

## Running Tests Manually

The PHP Extension Testing workflow can be run manually from the GitHub Actions tab, allowing you to test any PHP version on-demand without creating a PR.

---

## How to Run Manual Tests

### Step 1: Navigate to Actions Tab

1. Go to your repository on GitHub
2. Click on the **Actions** tab at the top
3. Find **PHP Extension Testing** in the left sidebar

### Step 2: Run Workflow

1. Click the **Run workflow** button (top right)
2. You'll see a form with two inputs:

   **PHP version to test:**
   - Enter the PHP version you want to test (e.g., `8.3.27`)
   - This version must exist in `releases.properties`
   - Required field

   **Platform to test on:**
   - Choose from dropdown:
     - `all` (default) - Tests on all 4 platforms
     - `win10-amd` - Windows 10 AMD only
     - `win10-intel` - Windows 10 Intel only
     - `win11-amd` - Windows 11 AMD only
     - `win11-intel` - Windows 11 Intel only

3. Click **Run workflow** button

---

## Viewing Results

### In the Actions Tab

All output is visible directly in the GitHub Actions interface:

1. **Workflow Run Summary**
   - Click on the workflow run to see the overview
   - Shows all jobs and their status

2. **Job Details**
   - Click on any job (e.g., "Test PHP 8.3.27 on win10-amd")
   - Expand each step to see detailed output
   - All `Write-Host` commands are visible here

3. **Visual Output**
   - ✅ Green checkmarks for passed steps
   - ❌ Red X for failed steps
   - ⚠️ Yellow warning for partial failures
   - Real-time logs as the workflow runs

### Example Output You'll See:

```
=== Phase 1.1: Download and Extract PHP 8.3.27 ===
Download URL: https://github.com/Bearsampp/module-php/releases/download/...
Downloading PHP archive...
Downloaded: php-8.3.27.7z (25.43 MB)
Extracting PHP archive...
✅ PHP extracted successfully
PHP executable: test-php-8.3.27\php.exe

=== Phase 1.2: Verify PHP Executable ===
Running: php -v
PHP 8.3.27 (cli) (built: Oct 31 2024 12:34:56) (ZTS Visual C++ 2019 x64)
✅ PHP version matches: 8.3.27
✅ PHP executes code successfully

=== Phase 2.1: Download Extensions ===
Downloading imagick...
✅ Downloaded: php_imagick-3.7.0-8.3-ts-vs16-x64.zip (234.56 KB)
✅ Extracted to: test-extensions\imagick

...and so on for all phases
```

### Artifacts

Even with manual runs, artifacts are still uploaded:

1. Go to the workflow run summary
2. Scroll to the bottom
3. Download artifacts like:
   - `test-results-8.3.27-win10-amd`
   - `test-results-8.3.27-win11-intel`
4. Contains JSON files with detailed test results

---

## Use Cases

### 1. Testing Existing Versions

Test a PHP version that's already in the repository:

```
PHP version: 8.3.27
Platform: all
```

This will run all 5 phases of testing across all platforms.

### 2. Quick Single-Platform Test

Test on just one platform to save time:

```
PHP version: 8.4.14
Platform: win10-amd
```

Useful for quick validation or debugging.

### 3. Debugging Failed Tests

If a PR test failed on a specific platform:

```
PHP version: 8.3.27
Platform: win11-intel
```

Run just that platform to investigate the issue.

### 4. Pre-PR Validation

Before creating a PR, test your new PHP version:

```
PHP version: 8.4.15
Platform: all
```

Ensure everything works before opening the PR.

---

## Differences from PR Tests

| Feature | PR Tests | Manual Tests |
|---------|----------|--------------|
| **Trigger** | Automatic on PR | Manual button click |
| **Version Detection** | Auto from git diff | You specify version |
| **Platform Selection** | Always all 4 | Choose specific or all |
| **PR Comments** | Yes, posts to PR | No (manual run) |
| **Artifacts** | Yes, uploaded | Yes, uploaded |
| **Visible Output** | Yes, in Actions | Yes, in Actions |

---

## Tips for Manual Testing

### 1. Check Version Exists

Before running, verify the version exists in `releases.properties`:

```bash
grep "^8.3.27" releases.properties
```

### 2. Test Incrementally

When debugging, test one platform at a time:
- Start with `win10-amd`
- If it passes, try `win11-amd`
- Then test Intel platforms

### 3. Watch Live Output

The Actions tab updates in real-time:
- Refresh the page to see progress
- Click on running jobs to see live logs
- No need to wait for completion

### 4. Compare Results

Run the same version on different platforms to compare:
- Download artifacts from each run
- Compare JSON files
- Identify platform-specific issues

### 5. Use for Development

When developing new test phases:
- Run manually to test your changes
- Iterate quickly without creating PRs
- Verify fixes before committing

---

## Example Workflow

### Scenario: Testing a New PHP Version

1. **Add version to releases.properties**
   ```properties
   8.4.15 = https://github.com/Bearsampp/module-php/releases/download/...
   ```

2. **Create bin directory structure**
   ```
   bin/php8.4.15/
   ├── exts.properties
   ├── deps.properties
   └── ...
   ```

3. **Run manual test first**
   - Go to Actions → PHP Extension Testing
   - Click "Run workflow"
   - Enter: `8.4.15`, Platform: `win10-amd`
   - Watch the output in real-time

4. **If successful, test all platforms**
   - Run again with Platform: `all`
   - Verify all 4 platforms pass

5. **Create PR**
   - Now confident it will pass
   - PR tests will run automatically
   - Results posted as comment

---

## Troubleshooting

### "Version not found in releases.properties"

**Problem:** The version you entered doesn't exist in the file.

**Solution:**
- Check spelling: `8.3.27` not `8.3.27.0`
- Ensure version is committed to your branch
- Verify the version line format is correct

### "No exts.properties file found"

**Problem:** The `bin/php{version}/` directory doesn't exist.

**Solution:**
- Create the directory structure
- Add required property files
- Commit and push changes

### "7-Zip is required"

**Problem:** GitHub Actions runner doesn't have 7z installed.

**Solution:**
- This shouldn't happen on GitHub-hosted runners
- If it does, the workflow will fail gracefully
- Check if the runner image changed

### Workflow Doesn't Appear

**Problem:** Can't find "PHP Extension Testing" in Actions tab.

**Solution:**
- Ensure the workflow file is committed
- Check file is in `.github/workflows/`
- Verify YAML syntax is correct
- Push to GitHub and refresh

---

## Advanced Usage

### Testing Multiple Versions

Run the workflow multiple times with different versions:

1. Run: `8.3.27`, Platform: `all`
2. Run: `8.4.14`, Platform: `all`
3. Compare results in Actions tab

### Platform-Specific Debugging

If one platform consistently fails:

1. Run just that platform: `win11-intel`
2. Download artifacts
3. Examine JSON files for errors
4. Check architecture validation results
5. Verify extension compatibility

### Regression Testing

Test older versions to ensure they still work:

```
PHP version: 8.1.32
Platform: all
```

Useful after making workflow changes.

---

## Summary

**Manual testing gives you:**
- ✅ Full control over what to test
- ✅ Real-time visual output in Actions tab
- ✅ Ability to test without creating PRs
- ✅ Platform-specific testing for debugging
- ✅ Quick iteration during development
- ✅ All the same validation as PR tests

**No PR comments are posted** for manual runs, but all output is visible in the Actions tab and artifacts are still uploaded for detailed analysis.

---

**Pro Tip:** Bookmark the Actions tab URL for quick access to manual testing!
