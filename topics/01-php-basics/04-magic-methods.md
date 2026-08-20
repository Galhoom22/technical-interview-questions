# What are Magic Methods in PHP, and when should they be used?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

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

- `__construct` — to put the object in a valid state when you use `new` (named factories are also valid).
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
- [PHP 8.5 new features](https://www.php.net/manual/en/migration85.new-features.php) — `clone($object, [...])` wither pattern
- [PHP 8.5 deprecated features](https://www.php.net/manual/en/migration85.deprecated.php) — `__sleep()` / `__wakeup()` are soft-deprecated; use `__serialize()` / `__unserialize()`

---

[← Previous](./03-array-types.md) · [Topic](./README.md)
