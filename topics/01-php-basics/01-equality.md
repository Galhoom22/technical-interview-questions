# What is the difference between `==` and `===`?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

`==` is **loose equality**. It compares values after PHP juggles types so both sides share a type.

`===` is **strict equality**. It compares value **and** type, with no conversion.

`==` asks “are these equal after conversion?”. `===` asks “are these the same thing?”.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Loose equality (`==`) | Compare values after PHP converts types so both sides match. |
| Strict equality (`===`) | Compare value **and** type. No conversion. Also called *identical*. |
| Type juggling | PHP changing a value’s type so `==` can compare (e.g. `"1"` → `1`). |
| `match` | PHP 8+ switch-like expression. Arms use `===`, not `==`. |

### 🧠 Analogy

`==` is squinting at two badges until they *look* the same. `===` is checking the badge **type** as well as the number. `"1"` and `1` pass the squint; they fail the type check.

| Aspect | `==` | `===` |
|--------|------|--------|
| Name | Loose equality | Strict equality |
| Compares | Values | Values **and** types |
| Type conversion | Yes | No |
| `1` vs `"1"` | `true` | `false` |
| `0` vs `false` | `true` | `false` |
| `null` vs `false` | `true` | `false` |
| `""` vs `0` | `true` | `false` |

---

### 📘 Official ([PHP Manual: Comparison Operators](https://www.php.net/manual/en/language.operators.comparison.php))

```php
var_dump(0 == "a");     // true in PHP 7, false in PHP 8 (string-to-number change)
var_dump("1" == "01");  // true — numeric strings compared numerically
var_dump(0 == false);   // true — bool/null are compared as bool
var_dump(1 === "1");    // false — identical: value and type, no juggling
```

### 💼 In production

```php
$status = $request->input('status'); // HTML/JSON often sends "1" or "paid"

if ($status == 1) {
    // also matches true, "1", "01" — a form checkbox can slip through
}

if ($order->status === 'paid') {
    // only the string paid
}

if ($order->user_id === $request->user()->id) {
    // IDs: always === so "12" never matches int 12 by accident
}
```

Loose comparison is dangerous because the conversion rules are easy to get wrong, especially with `0`, `""`, `null`, `false`, and numeric strings. PHP 8 tightened some of those rules, but relying on them is still a trap. `===` never surprises you.

**Practical points for the interview:**

- Default to `===` / `!==`. Use `==` only when you *intentionally* want type coercion (rare).
- `switch` uses `==`. `match` uses `===`. That is one reason `match` is safer.
- `in_array($value, $list)` uses `==` by default. Pass `true` as the third argument for strict comparison: `in_array($value, $list, true)`.
- Same idea for `array_search()` and `array_keys()` with the strict flag.
- If you need “is this empty?”, do not use `== null` or `== false`. Use `=== null`, `=== ''`, or `empty()` / `isset()` depending on the intent.

---

### ⚠️ Watch out

> [!WARNING]
> `switch` and `in_array()` are loose (`==`) unless you opt into strict. `0`, `""`, `null`, and `"1"` are where people fail this question.

### 💬 If they follow up

> [!NOTE]
> - Why did `0 == "foo"` change? PHP 8 stopped treating non-numeric strings as `0` in that comparison. Still prefer `===`.
> - `match` vs `switch`? `match` uses `===`. That is one reason it is safer.

---

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

[Topic](./README.md) · [Next →](./02-array-types.md)