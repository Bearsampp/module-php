# Documentation Fixes Needed

## Issue: References to `build/distributions` Should Be Removed

The documentation incorrectly references `build/distributions/` which is NOT used by the build system. The build uses external temp folders in `bearsampp-build/` instead, matching the MySQL module pattern.

---

## Files That Need Fixing

### 1. `.gradle-docs/README.md`

**Lines to Fix**: ~195-210 (Directory Structure section)

**Current (WRONG)**:
```
├── tmp/                   # Temporary build files
│   ├── prep/              # Prepared bundles
│   └── downloads/         # Downloaded dependencies
├── build/                 # Gradle build outputs
│   └── distributions/     # Final packaged archives (.7z/.zip)
```

**Should Be**:
```
bearsampp-build/                    # External build directory (outside repo)
├── tmp/                            # Temporary build files
│   ├── bundles_prep/bins/php/      # Prepared bundles
│   ├── bundles_build/bins/php/     # Build staging
│   ├── downloads/php/              # Downloaded dependencies
│   └── extract/php/                # Extracted archives
└── bins/php/                       # Final packaged archives
    └── 2025.10.31/                 # Release version
        ├── bearsampp-php-8.3.15-2025.10.31.7z
        ├── bearsampp-php-8.3.15-2025.10.31.7z.md5
        └── ...
```

**Line ~243**: Build process flow

**Current (WRONG)**:
```
9. Package prepared folder into archive in build/distributions/
```

**Should Be**:
```
9. Package prepared folder into archive in bearsampp-build/bins/php/{bundle.release}/
```

---

### 2. `.gradle-docs/TASKS.md`

**Lines to Fix**: Multiple references

**Search for**: `build/distributions`
**Replace with**: `bearsampp-build/bins/php/{bundle.release}`

**Search for**: `tmp/prep`
**Replace with**: `bearsampp-build/tmp/bundles_prep/bins/php`

---

### 3. `.gradle-docs/INDEX.md`

**Line ~188**:

**Current (WRONG)**:
```
| `tmp/`                | Temporary build files                    |
```

**Should Be**:
```
| `bearsampp-build/tmp/` | Temporary build files (external)       |
| `bearsampp-build/bins/` | Final packaged archives (external)    |
```

---

## Summary of Changes

### What's Wrong
- Documentation references `build/distributions/` which doesn't exist
- Documentation references `tmp/` in project root, but it's actually in `bearsampp-build/tmp/`
- Missing information about external `bearsampp-build/` directory

### What's Correct (in build.gradle)
- Uses `bearsampp-build/tmp/` for all temporary files
- Uses `bearsampp-build/bins/php/{bundle.release}/` for final archives
- Matches MySQL module pattern exactly

### Why This Matters
- Users will look in wrong location for build outputs
- Misleading about where temporary files are stored
- Doesn't explain the external build directory structure

---

## Quick Fix Commands

Since the special characters in the files make automated replacement difficult, here's what needs to be manually updated:

1. **README.md** - Update Directory Structure section (lines ~195-210)
2. **README.md** - Update Build Process Flow step 9 (line ~243)
3. **TASKS.md** - Replace all `build/distributions` with `bearsampp-build/bins/php/{bundle.release}`
4. **TASKS.md** - Replace all `tmp/prep` with `bearsampp-build/tmp/bundles_prep/bins/php`
5. **INDEX.md** - Update directory table (line ~188)

---

## Correct Directory Structure (For Reference)

```
Project Layout:
===============

module-php/                         # Git repository
├── bin/                            # PHP versions (in repo)
├── pear/                           # PEAR scripts (in repo)
├── build.gradle                    # Build script
└── ...

bearsampp-build/                    # External (NOT in repo)
├── tmp/                            # All temporary files
│   ├── bundles_prep/bins/php/      # Prepared bundles
│   │   └── php8.3.15/              # Prepared PHP version
│   ├── bundles_build/bins/php/     # Build staging
│   ├── downloads/php/              # Downloaded files
│   └── extract/php/                # Extracted archives
└── bins/php/                       # Final outputs
    └── 2025.10.31/                 # Release version folder
        ├── bearsampp-php-8.3.15-2025.10.31.7z
        ├── bearsampp-php-8.3.15-2025.10.31.7z.md5
        ├── bearsampp-php-8.3.15-2025.10.31.7z.sha1
        ├── bearsampp-php-8.3.15-2025.10.31.7z.sha256
        └── bearsampp-php-8.3.15-2025.10.31.7z.sha512
```

This matches the MySQL module pattern exactly.
