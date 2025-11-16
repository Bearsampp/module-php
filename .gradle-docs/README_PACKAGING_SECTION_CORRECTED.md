# Corrected Packaging Details Section for README.md

## Replace lines ~247-270 in .gradle-docs/README.md with:

```markdown
### Packaging Details

- **Archive name format**: `bearsampp-php-{version}-{bundle.release}.{7z|zip}`
- **Location**: `bearsampp-build/bins/php/{bundle.release}/`
  - Example: `bearsampp-build/bins/php/2025.10.31/bearsampp-php-8.3.15-2025.10.31.7z`
- **Content root**: The top-level folder inside the archive is `php{version}/` (e.g., `php8.3.15/`)
- **Structure**: The archive contains the PHP version folder at the root with all PHP files inside

**Archive Structure Example**:
```
bearsampp-php-8.3.15-2025.10.31.7z
└── php8.3.15/              ← Version folder at root
    ├── php.exe
    ├── php-cgi.exe
    ├── php.ini
    ├── ext/                ← Extensions directory
    │   ├── php_curl.dll
    │   ├── php_mbstring.dll
    │   └── ...
    ├── pear/               ← PEAR installation (if configured)
    └── ...
```

**Verification Commands**:

```bash
# List archive contents with 7z
7z l bearsampp-build/bins/php/2025.10.31/bearsampp-php-8.3.15-2025.10.31.7z | more

# You should see entries beginning with:
#   php8.3.15/php.exe
#   php8.3.15/php-cgi.exe
#   php8.3.15/ext/php_curl.dll
#   php8.3.15/...

# Extract and inspect with PowerShell (zip example)
Expand-Archive -Path bearsampp-build/bins/php/2025.10.31/bearsampp-php-8.3.15-2025.10.31.zip -DestinationPath .\_inspect
Get-ChildItem .\_inspect\php8.3.15 | Select-Object Name

# Expected output:
#   php.exe
#   php-cgi.exe
#   php.ini
#   ext/
#   ...
```

**Note**: This archive structure matches the MySQL module pattern where archives contain `mysql{version}/` at the root. This ensures consistency across all Bearsampp modules.

**Hash Files**: Each archive is accompanied by hash sidecar files:
- `.md5` - MD5 checksum
- `.sha1` - SHA-1 checksum
- `.sha256` - SHA-256 checksum
- `.sha512` - SHA-512 checksum

Example:
```
bearsampp-build/bins/php/2025.10.31/
├── bearsampp-php-8.3.15-2025.10.31.7z
├── bearsampp-php-8.3.15-2025.10.31.7z.md5
├── bearsampp-php-8.3.15-2025.10.31.7z.sha1
├── bearsampp-php-8.3.15-2025.10.31.7z.sha256
└── bearsampp-php-8.3.15-2025.10.31.7z.sha512
```
```

## Why This Correction is Needed

The original documentation stated:
- Content root: `{bundle.release}/` (e.g., `2025.10.31/`)
- Inside that: `bin/php{version}/...`

But the actual code creates:
- Content root: `php{version}/` (e.g., `php8.3.15/`)
- Directly contains PHP files

This matches the MySQL module pattern and is the correct behavior.
