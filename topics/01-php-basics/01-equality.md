# What is the difference between `==` and `===`?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

`==` is **loose equality**. It compares values after PHP juggles types so both sides share a type.

`===` is **strict equality**. It compares value **and** type, with no conversion.

`==` asks “are these equal after conversion?”. `===` asks “are these the same thing?”.

| Aspect | `==` | `===` |
|--------|------|--------|
| Name | Loose equality | Strict equality |
| Compares | Values | Values **and** types |
| Type conversion | Yes | No |
| `1` vs `"1"` | `true` | `false` |
| `0` vs `false` | `true` | `false` |
| `null` vs `false` | `true` | `false` |
| `""` vs `0` | `true` | `false` |

```php
1 == "1";   // true  — "1" is converted to int 1
1 === "1";  // false — int vs string

0 == false;   // true
0 === false;  // false — int vs bool

null == false;   // true
null === false;  // false

"" == 0;   // true
"" === 0;  // false

// PHP 8.0+: match is strict (===). switch is loose (==).
echo match (1) {
    '1' => 'skipped — match uses ===',
    1 => 'this arm runs',
};
```

Loose comparison is dangerous because the conversion rules are easy to get wrong, especially with `0`, `""`, `null`, `false`, and numeric strings:

```php
"10abc" == 10;  // true in PHP 7 (string becomes 10), false in PHP 8
0 == "foo";     // true in PHP 7, false in PHP 8
```

PHP 8 tightened some of those rules, but relying on them is still a trap. `===` never surprises you.

**Practical points for the interview:**

- Default to `===` / `!==`. Use `==` only when you *intentionally* want type coercion (rare).
- `switch` uses `==`. `match` uses `===`. That is one reason `match` is safer.
- `in_array($value, $list)` uses `==` by default. Pass `true` as the third argument for strict comparison: `in_array($value, $list, true)`.
- Same idea for `array_search()` and `array_keys()` with the strict flag.
- If you need “is this empty?”, do not use `== null` or `== false`. Use `=== null`, `=== ''`, or `empty()` / `isset()` depending on the intent.

> [!TIP]
> **One-liner:** `==` compares values with type juggling; `===` compares values and types with no juggling. Prefer `===` unless you have a specific reason not to.

**Source:** [PHP: Comparison Operators](https://www.php.net/manual/en/language.operators.comparison.php) — official definition of `==` (equal, type juggling) vs `===` (identical), including the PHP 8 string-to-number change.

**Learn more:**
- [PHP type juggling](https://www.php.net/manual/en/language.types.type-juggling.php) — how `==` converts types
- [PHP type comparison tables](https://www.php.net/manual/en/types.comparisons.php) — full `==` / `===` matrix for every type pair
- [The `match` expression](https://www.php.net/manual/en/control-structures.match.php) — `match` uses identity (`===`); `switch` uses equality (`==`)
- [`in_array()`](https://www.php.net/manual/en/function.in-array.php) — third argument `$strict` for the same trap in arrays
- [`array_search()`](https://www.php.net/manual/en/function.array-search.php) — same `$strict` flag; default is loose `==`
- [`empty()`](https://www.php.net/manual/en/function.empty.php) / [`isset()`](https://www.php.net/manual/en/function.isset.php) — do not fake these with `== null`

---

[Topic](./README.md) · [Next →](./02-require-vs-include.md)
