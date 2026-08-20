# 🐘 PHP - Basics

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
```

You can mix numeric and string keys in the same array. Internally PHP still stores it as one hash table (plus a packed list optimization for pure 0-based integer keys).

**Practical points for the interview:**

- Prefer short syntax `[]` over `array()`.
- For a list of models/DTOs I use a **list** (indexed). For a record of named fields I use **associative**.
- Nested arrays get messy fast. After one or two levels I switch to objects / DTOs / collections.
- PHP 8.1+ `array_is_list()` tells you whether keys are `0..n-1` with no gaps.

> [!TIP]
> **One-liner:** PHP has one array type used in three ways — indexed, associative, and multidimensional (arrays of arrays).

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
| `__clone()` | The object is cloned |
| `__serialize()` / `__unserialize()` | `serialize()` / `unserialize()` (preferred over `__sleep` / `__wakeup`) |
| `__debugInfo()` | The object is dumped with `var_dump()` |

```php
class Money
{
    public function __construct(
        public readonly int $cents,
        public readonly string $currency,
    ) {}

    public function __toString(): string
    {
        return ($this->cents / 100) . ' ' . $this->currency;
    }
}

echo new Money(1999, 'USD'); // "19.99 USD"
```

**When to use them:**

- `__construct` — always, for a valid object.
- `__toString` / `__invoke` — when the object has a natural string or callable form.
- `__serialize` — when you persist or queue the object.
- `__clone` — when a shallow copy is not enough (nested objects).

**When not to use them:**

- Do not replace a real public API with `__get` / `__set` / `__call`. They hide typos, break static analysis, and are slower.
- Do not put business logic in `__destruct`. Shutdown order is not something you want to depend on.

> [!TIP]
> **One-liner:** Magic methods are hooks PHP runs for you (`__construct`, `__toString`, `__get`, …). Use them for construction, string form, and serialization — not as a hidden public API.
