# What are the different types of arrays in PHP?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

PHP has **one** array type: an ordered map. Interviewers still expect the three ways we *use* that type:

**Key terms:**

| Term | Plain meaning |
|------|----------------|
| Array | PHP’s one array type: an ordered map. Keys are `int` or `string`. |
| Indexed array | Keys `0`, `1`, `2`, … — a list. |
| Associative array | String keys — a record of named fields. |
| Multidimensional array | Arrays inside arrays (a list of records, a tree, …). |
| `array_is_list()` | PHP 8.1+: `true` if keys are `0..n-1` with no gaps. |

**Analogy:**

PHP has one toolbox (the array). You use it as a numbered list, a labeled drawer, or a drawer of drawers. Same tool, three jobs.

| Type | Keys | Example |
|------|------|---------|
| Indexed (numeric) | `0`, `1`, `2`, … | `['php', 'laravel']` |
| Associative | String keys | `['name' => 'Omar', 'role' => 'backend']` |
| Multidimensional | Arrays inside arrays | `$users[0]['email']` |

**Official** ([PHP Manual: Arrays](https://www.php.net/manual/en/language.types.array.php)):

```php
$array1 = array(
    "foo" => "bar",
    "bar" => "foo",
);

$array2 = [
    "foo" => "bar",
    "bar" => "foo",
];
```

Same type either way: an ordered map. Keys may be `int` or `string`; values may be anything, including more arrays.

**In production:**

```php
$skuList = ['TSHIRT-S', 'TSHIRT-M', 'TSHIRT-L']; // indexed list of SKUs

$totals = [
    'subtotal_cents' => 4999,
    'tax_cents' => 700,
    'currency' => 'USD',
];

$checkout = [
    ['sku' => 'TSHIRT-M', 'qty' => 2, 'unit_cents' => 1999],
    ['sku' => 'MUG', 'qty' => 1, 'unit_cents' => 1299],
];

array_is_list($skuList);     // true
array_first($checkout);      // first line item
array_last($totals);         // 'USD'
```

You can mix numeric and string keys in the same array. Internally PHP still stores it as one hash table (plus a packed list optimization for pure 0-based integer keys).

**Practical points for the interview:**

- Prefer short syntax `[]` over `array()`.
- For a list of models/DTOs I use a **list** (indexed). For a record of named fields I use **associative**.
- Nested arrays get messy fast. After one or two levels I switch to objects / DTOs / collections.
- PHP 8.1+ `array_is_list()` tells you whether keys are `0..n-1` with no gaps.
- PHP 8.5 `array_first()` / `array_last()` return the first or last value (or `null` if empty).
- PHP 8.5 pipe operator `|>` is useful for transforming values without nested calls.

**Watch out:**

The key `"8"` is stored as int `8`. Mixing list keys and string keys in one array is legal and usually a mess. Deep nested arrays are a smell — switch to objects/DTOs.

**If they follow up:**

- How do you detect a list? `array_is_list()` (PHP 8.1+).
- `array_first` / `array_last`? PHP 8.5 value helpers (older: `array_key_first` / `array_key_last`).

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
