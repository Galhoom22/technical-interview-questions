# What is the difference between `require` and `include` in PHP?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Both load and execute another PHP file in the current scope. The difference is **what happens if the file is missing**.

| Aspect | `require` / `require_once` | `include` / `include_once` |
|--------|----------------------------|----------------------------|
| Missing file | Fatal error — script **stops** | Warning — script **continues** |
| Use when | The file is mandatory | The file is optional |
| Typical files | Autoload, config, core classes | Optional template / partial |

```php
require __DIR__ . '/config.php';   // app cannot run without this
include __DIR__ . '/banner.php';   // nice to have; failure is not fatal
```

`require_once` and `include_once` do the same thing, but skip the file if it was already loaded. That prevents “cannot redeclare function/class” errors.

**Practical points for the interview:**

- For anything the app cannot boot without, I use `require` / `require_once`.
- Always build the path from `__DIR__` (or Composer autoload). A bare relative path depends on the current working directory, which is easy to get wrong.
- In modern PHP I almost never `require` class files by hand — Composer’s PSR-4 autoloader does that. `require` still matters for `vendor/autoload.php`, config, and a few bootstrap files.
- `include` returning `false` on failure is not a substitute for handling errors. If the file is required for correctness, use `require`.

> [!TIP]
> **One-liner:** `require` kills the script if the file is missing; `include` only warns and continues. Use `require` for files the app cannot run without.

**Source:** [`require`](https://www.php.net/manual/en/function.require.php) and [`include`](https://www.php.net/manual/en/function.include.php) — official PHP manual; this answer follows those two pages.

**Learn more:**
- [`require_once`](https://www.php.net/manual/en/function.require-once.php) / [`include_once`](https://www.php.net/manual/en/function.include-once.php) — skip a file that was already loaded
- [Composer: Autoloading](https://getcomposer.org/doc/01-basic-usage.md#autoloading) — the current standard way to load classes
- [PSR-4: Autoloader](https://www.php-fig.org/psr/psr-4/) — the spec Composer implements
- [PHP: Autoloading Classes](https://www.php.net/manual/en/language.oop5.autoload.php) — `spl_autoload_register` and how `new` triggers a file load
- [PSR-12: Extended Coding Style](https://www.php-fig.org/psr/psr-12/) — PHP-FIG coding-style PSR
- [PER Coding Style](https://www.php-fig.org/per/coding-style/) — living PHP-FIG style (Pint’s `per` preset)

---

[← Previous](./01-equality.md) · [Topic](./README.md) · [Next →](./03-array-types.md)
