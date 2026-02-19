# CLAUDE.md

## Project Overview

`phpstan/build-cs` is a shared PHP CodeSniffer (PHPCS) configuration used across all PHPStan extension repositories. It provides a centralized coding standard so that every PHPStan extension enforces the same style and quality rules.

This repository does **not** contain PHP source code itself. It contains PHPCS ruleset XML files and the Composer dependencies needed to run them.

## Repository Structure

- `phpcs.xml` - Main PHPCS ruleset ("PHPStan Extensions Coding Standard"). This is the entry point that extension repositories reference when running coding standard checks.
- `consistence.xml` - Base ruleset derived from the Consistence coding standard. Referenced by `phpcs.xml` with some rules excluded or reconfigured.
- `composer.json` / `composer.lock` - Composer dependencies (PHP_CodeSniffer 4.x, Slevomat Coding Standard 8.x).
- `.github/workflows/build.yml` - CI workflow that validates the coding standard against all PHPStan extension repositories.
- `.github/renovate.json` - Renovate bot configuration for automated dependency updates.
- `tmp/` - Cache directory for PHPCS (contents git-ignored).

## How It Works

The CI workflow in `build.yml` tests this coding standard against 13 PHPStan repositories:

1. Checks out the target extension repository
2. Checks out `build-cs` into a `build-cs/` subdirectory
3. Installs `build-cs` Composer dependencies
4. Runs `make cs` in the extension repository

Each extension repository has its own `Makefile` with a `cs` target that runs PHPCS using the ruleset from `build-cs/phpcs.xml`.

### Tested Repositories

The coding standard is validated against: `phpstan-doctrine`, `extension-installer`, `phpstan-phpunit`, `phpstan-mockery`, `phpstan-symfony`, `phpdoc-parser`, `phpstan-nette`, `phpstan-webmozart-assert`, `phpstan-beberlei-assert`, `phpstan-deprecation-rules`, `phpstan-dibi`, `phpstan-strict-rules`, and `phpstan-src`.

## PHP Version Compatibility

The coding standard targets **PHP 7.4+** (`php_version` is set to `70400` in `phpcs.xml`). While `phpstan-src` itself uses PHP 8.1+ (thanks to PHAR build downgrade), other PHPStan extension repositories still need to support PHP 7.4+. This means:

- Trailing commas in closure `use` lists and function declarations are **disallowed** (these are PHP 8.0+ syntax).
- Trailing commas in function/method **calls** are required (PHP 7.3+ syntax).
- Arrow functions are required where possible (PHP 7.4+ syntax).

## Key Coding Standard Rules

### Formatting
- **Indentation**: Tabs (not spaces)
- **Brace style**: BSD/Allman style for functions (opening brace on its own line)
- **Line endings**: Unix (`\n`)
- **Encoding**: UTF-8
- **`declare(strict_types=1)`**: Required on the first line of every file

### Naming and Namespaces
- Namespace `PHPStan` is mapped to `src/` and `tests/` directories (type name must match file name)
- `use` statements must be alphabetically sorted (case-insensitive, PSR-12 compatible)
- No group `use` declarations
- No unused `use` statements
- Global functions and constants must be explicitly imported (no fallback)
- No superfluous `Abstract` prefix on abstract classes or `Interface` suffix on interfaces

### Type Hints
- Parameter, property, and return type hints are enforced
- Useless PHPDoc annotations (that just duplicate native type hints) are flagged
- `null` type hint must appear last in union types

### Code Quality
- Early exit pattern is encouraged
- Static closures required where possible
- Strict comparison operators required (`===`/`!==`, not `==`/`!=`)
- No Yoda comparisons
- No `empty()` usage
- Combined assignment operators required where applicable
- Null coalesce operator (`??`) and null coalesce assignment (`??=`) required where applicable
- Unused variables are flagged
- Variables in double-quoted strings are disallowed (use `sprintf()` instead)

### Excluded Patterns
- `tests/*/data` and `tests/*/data-attributes` directories are excluded from checks (these contain test fixture files in the extension repositories)

## Branching

- The main branch is **`2.x`**.

## Making Changes

When modifying the coding standard rules:

1. Edit `phpcs.xml` for PHPStan-specific rule configuration, or `consistence.xml` for base rules.
2. The CI will automatically test changes against all PHPStan extension repositories to catch any regressions.
3. Keep PHP 7.4 compatibility in mind - do not enable rules that require PHP 8.0+ syntax in extension code.
