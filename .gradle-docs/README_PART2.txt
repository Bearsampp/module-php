### Extension Processing

Each PHP version can have an `exts.properties` file:

```properties
phpexts.imagick=https://windows.php.net/downloads/pecl/releases/imagick/3.7.0/php_imagick-3.7.0-8.3-ts-vs16-x64.zip
phpexts.redis=https://windows.php.net/downloads/pecl/releases/redis/6.0.2/php_redis-6.0.2-8.3-ts-vs16-x64.zip
phpexts.xdebug=https://xdebug.org/files/php_xdebug-3.3.1-8.3-vs16-x86_64.dll
```

### PEAR Processing

PEAR configuration in `pear.properties`:

```properties
phppear.pear=https://pear.php.net/install-pear-nozlib.phar
```

### Dependency Processing

Dependencies in `deps.properties`:

```properties
phpdeps.imagemagick=https://imagemagick.org/archive/binaries/ImageMagick-7.1.1-28-portable-Q16-HDRI-x64.zip
```

---

## Troubleshooting

### Common Issues

#### Issue: "Dev path not found"

**Symptom:**
```
Dev path not found: E:/Bearsampp-development/dev
```

**Solution:**
This is a warning only. The dev path is optional for most tasks. If you need it, ensure the `dev` project exists in the parent directory.

---

#### Issue: "Bundle version not found"

**Symptom:**
```
Bundle version not found: E:/Bearsampp-development/module-php/bin/php8.3.99
```

**Solution:**
1. List available versions: `gradle listVersions`
2. Use an existing version: `gradle release -PbundleVersion=8.3.15`

---

#### Issue: "DLL is not 64-bit architecture"

**Symptom:**
```
DLL php_extension.dll is not 64-bit architecture. Build halted.
```

**Solution:**
The extension DLL is not 64-bit compatible. Download the correct 64-bit version from the extension provider.

---

#### Issue: "Java version too old"

**Symptom:**
```
Java 8+ required
```

**Solution:**
1. Check Java version: `java -version`
2. Install Java 8 or higher
3. Update JAVA_HOME environment variable

---

### Debug Mode

Run Gradle with debug output:

```bash
gradle release -PbundleVersion=8.3.15 --info
gradle release -PbundleVersion=8.3.15 --debug
```

### Clean Build

If you encounter issues, try a clean build:

```bash
gradle clean
gradle release -PbundleVersion=8.3.15
```

---

## Migration Guide

### From Ant to Gradle

The project has been fully migrated from Ant to Gradle. Here's what changed:

#### Removed Files

| File              | Status    | Replacement                |
|-------------------|-----------|----------------------------|
| `build.xml`       | âŒ Removed | `build.gradle`             |
| `sigcheck.xml`    | âŒ Removed | Integrated into build.gradle |

#### Command Mapping

| Ant Command                          | Gradle Command                              |
|--------------------------------------|---------------------------------------------|
| `ant release`                        | `gradle release`                            |
| `ant release -Dinput.bundle=8.3.15`  | `gradle release -PbundleVersion=8.3.15`     |
| `ant clean`                          | `gradle clean`                              |

#### Key Differences

| Aspect              | Ant                          | Gradle                           |
|---------------------|------------------------------|----------------------------------|
| **Build File**      | XML (build.xml)              | Groovy DSL (build.gradle)        |
| **Task Definition** | `<target name="...">`        | `tasks.register('...')`          |
| **Properties**      | `<property name="..." />`    | `ext { ... }`                    |
| **Dependencies**    | Manual downloads             | Automatic with repositories      |
| **Caching**         | None                         | Built-in incremental builds      |
| **IDE Support**     | Limited                      | Excellent (IntelliJ, Eclipse)    |

---

## Additional Resources

- [Gradle Documentation](https://docs.gradle.org/)
- [Bearsampp Project](https://github.com/bearsampp/bearsampp)
- [PHP Downloads](https://windows.php.net/download/)
- [PECL Extensions](https://windows.php.net/downloads/pecl/)

---

## Support

For issues and questions:

- **GitHub Issues**: https://github.com/bearsampp/module-php/issues
- **Bearsampp Issues**: https://github.com/bearsampp/bearsampp/issues
- **Documentation**: https://bearsampp.com/module/php

---

**Last Updated**: 2025-01-31  
**Version**: 2025.10.31  
**Build System**: Pure Gradle (no wrapper, no Ant)

Notes:
- This project deliberately does not ship the Gradle Wrapper. Install Gradle 8+ locally and run with `gradle ...`.
- Legacy Ant files (e.g., Eclipse `.launch` referencing `build.xml`) are deprecated and not used by the build.
