# 🐘 PHP - Basics

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

## Q1. What is the difference between `==` and `===`?

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

// PHP 8.5: match is strict (===). switch is loose (==).
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
- [PHP type comparison tables](https://www.php.net/manual/en/types.comparisons.php) — full `==` / `===` matrix for every type pair
- [The `match` expression](https://www.php.net/manual/en/control-structures.match.php) — `match` uses identity (`===`); `switch` uses equality (`==`)
- [`in_array()`](https://www.php.net/manual/en/function.in-array.php) — third argument `$strict` for the same trap in arrays

---

## Q2. What is the difference between `require` and `include` in PHP?

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
- [PSR-4: Autoloader](https://www.php-fig.org/psr/psr-4/) — how Composer loads classes so you rarely `require` them by hand
- [PHP: Autoloading Classes](https://www.php.net/manual/en/language.oop5.autoload.php) — `spl_autoload_register` and how `new` triggers a file load

---

## Q3. What are the different types of arrays in PHP?

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
- PHP 8.5 `array_is_list()` tells you whether keys are `0..n-1` with no gaps.
- PHP 8.5 `array_first()` / `array_last()` return the first or last value (or `null` if empty).
- PHP 8.5 pipe operator `|>` is useful for transforming list values without nested calls.

> [!TIP]
> **One-liner:** PHP has one array type used in three ways — indexed, associative, and multidimensional (arrays of arrays).

**Source:** [PHP: Arrays](https://www.php.net/manual/en/language.types.array.php) — official type page: PHP arrays are ordered maps (this is the “one type, three usages” answer).

**Learn more:**
- [`array_is_list()`](https://www.php.net/manual/en/function.array-is-list.php) — detect a packed `0..n-1` list vs a map
- [`array_first()`](https://www.php.net/manual/en/function.array-first.php) / [`array_last()`](https://www.php.net/manual/en/function.array-last.php) — PHP 8.5 helpers
- [PHP 8.5 new features](https://www.php.net/manual/en/migration85.new-features.php) — pipe operator `|>` and more

---

## Q4. What are Magic Methods in PHP, and when should they be used?

**Answer:**

Magic methods are methods PHP calls **automatically** when a certain action happens. Their names start with `__`.

You do not call most of them yourself. The engine does.

| Method | PHP calls it when… |
|--------|---------------------|
| `__construct()` | The object is created with `new` |
| `__destruct()` | The object is destroyed (no more references, or shutdown) |
| `__get()` / `__set()` | Reading / writing an inaccessible property |
| `__isset()` / `__unset()` | `isset()` / `unset()` on an inaccessible property |
| `__call()` / `__callStatic()` | Calling an inaccessible instance / static method |
| `__toString()` | The object is used as a string |
| `__invoke()` | The object is called like a function `$obj()` |
| `__clone()` | The object is cloned (`clone $obj`, or PHP 8.5 `clone($obj, [...])`) |
| `__serialize()` / `__unserialize()` | `serialize()` / `unserialize()` (`__sleep` / `__wakeup` are soft-deprecated in PHP 8.5) |
| `__debugInfo()` | The object is dumped with `var_dump()` |

```php
readonly class Money
{
    public function __construct(
        public int $cents,
        public string $currency,
    ) {}

    public function __toString(): string
    {
        return ($this->cents / 100) . ' ' . $this->currency;
    }

    public function withCents(int $cents): self
    {
        return clone($this, ['cents' => $cents]);
    }
}

echo new Money(1999, 'USD'); // "19.99 USD"
```

**When to use them:**

- `__construct` — always, for a valid object.
- `__toString` / `__invoke` — when the object has a natural string or callable form.
- `__serialize` — when you persist or queue the object.
- `__clone` / PHP 8.5 `clone($object, $withProperties)` — wither pattern for `readonly` classes.

**When not to use them:**

- Do not replace a real public API with `__get` / `__set` / `__call`. They hide typos, break static analysis, and are slower.
- Do not put business logic in `__destruct`. Shutdown order is not something you want to depend on.

> [!TIP]
> **One-liner:** Magic methods are hooks PHP runs for you (`__construct`, `__toString`, `__get`, …). Use them for construction, string form, and serialization — not as a hidden public API.

**Source:** [PHP: Magic Methods](https://www.php.net/manual/en/language.oop5.magic.php) — official list and contracts for every `__*` method this answer names.

**Learn more:**
- [Constructors and Destructors](https://www.php.net/manual/en/language.oop5.decon.php) — `__construct` / `__destruct` in full
- [Overloading](https://www.php.net/manual/en/language.oop5.overloading.php) — `__get`, `__set`, `__call`, `__callStatic` (and why they hide bugs)
- [PHP 8.5 new features](https://www.php.net/manual/en/migration85.new-features.php) — `clone($object, [...])` wither pattern; `__sleep` / `__wakeup` soft-deprecated
