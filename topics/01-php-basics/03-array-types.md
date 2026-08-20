# What are the different types of arrays in PHP?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

PHP has **one** array type: an ordered map. Interviewers still expect the three ways we *use* that type:

| Type | Keys | Example |
|------|------|---------|
| Indexed (numeric) | `0`, `1`, `2`, … | `['php', 'laravel']` |
| Associative | String keys | `['name' => 'Omar', 'role' => 'backend']` |
| Multidimensional | Arrays inside arrays | `$users[0]['email']` |

```php
$indexed = ['php', 'laravel', 'mysql'];

$associative = [
    'name' => 'Omar',
    'role' => 'backend',
];

$multidimensional = [
    ['id' => 1, 'name' => 'Omar'],
    ['id' => 2, 'name' => 'Sara'],
];

array_is_list($indexed);        // true
array_first($indexed);          // 'php'
array_last($associative);       // 'backend'

$slug = ' PHP 8.5 '
    |> trim(...)
    |> strtolower(...);
```

You can mix numeric and string keys in the same array. Internally PHP still stores it as one hash table (plus a packed list optimization for pure 0-based integer keys).

**Practical points for the interview:**

- Prefer short syntax `[]` over `array()`.
- For a list of models/DTOs I use a **list** (indexed). For a record of named fields I use **associative**.
- Nested arrays get messy fast. After one or two levels I switch to objects / DTOs / collections.
- PHP 8.1+ `array_is_list()` tells you whether keys are `0..n-1` with no gaps.
- PHP 8.5 `array_first()` / `array_last()` return the first or last value (or `null` if empty).
- PHP 8.5 pipe operator `|>` is useful for transforming values without nested calls.

> [!TIP]
> **One-liner:** PHP has one array type used in three ways — indexed, associative, and multidimensional (arrays of arrays).

**Source:** [PHP: Arrays](https://www.php.net/manual/en/language.types.array.php) — official type page: PHP arrays are ordered maps (this is the “one type, three usages” answer).

**Learn more:**
- [`array_is_list()`](https://www.php.net/manual/en/function.array-is-list.php) — detect a packed `0..n-1` list vs a map
- [`array_first()`](https://www.php.net/manual/en/function.array-first.php) / [`array_last()`](https://www.php.net/manual/en/function.array-last.php) — PHP 8.5 helpers
- [`array_key_first()`](https://www.php.net/manual/en/function.array-key-first.php) / [`array_key_last()`](https://www.php.net/manual/en/function.array-key-last.php) — PHP 7.3+ key helpers; 8.5 adds value helpers `array_first()` / `array_last()`
- [PHP 8.5 new features](https://www.php.net/manual/en/migration85.new-features.php) — pipe operator `|>` and more

---

[← Previous](./02-require-vs-include.md) · [Topic](./README.md) · [Next →](./04-magic-methods.md)
