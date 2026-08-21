# What are Magic Methods in PHP, and when should they be used?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Magic methods are methods PHP calls **automatically** when a certain action happens. Their names start with `__`.

You do not call most of them yourself. The engine does.

**🔑 Key terms:**

| Term | Plain meaning |
|------|----------------|
| Magic method | A `__*` method PHP calls for you when something happens (`new`, `echo $obj`, `clone`, …). |
| Constructor | `__construct` — runs after the engine creates the object, to initialize it. |
| Destructor | `__destruct` — runs when the object is destroyed. For resource cleanup, not business logic. |
| `__toString` | How the object looks when used as a string (`echo $obj`). |
| Overloading | `__get` / `__set` / `__call` — intercept missing properties/methods. Easy to hide bugs; use sparingly. |

**🧠 Analogy:**

Magic methods are automatic doors: they open when you walk up (`echo $obj`, `new`, `clone`). You do not push a button named `__toString`.

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

**📘 Official** ([PHP Manual: Magic Methods](https://www.php.net/manual/en/language.oop5.magic.php) — `__toString`):

```php
class TestClass
{
    public function __construct(public $foo) {}

    public function __toString()
    {
        return $this->foo;
    }
}

$class = new TestClass('Hello');
echo $class; // Hello — PHP called __toString(), not you
```

**💼 In production:**

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

echo new Money(1999, 'USD'); // "19.99 USD" on an invoice line
$discounted = (new Money(1999, 'USD'))->withCents(1499);
```

**When to use them:**

- `__construct` — to put the object in a valid state when you use `new` (named factories are also valid).
- `__toString` / `__invoke` — when the object has a natural string or callable form.
- `__serialize` — when you persist or queue the object.
- `__clone` / PHP 8.5 `clone($object, $withProperties)` — wither pattern for `readonly` classes.

**When not to use them:**

- Do not replace a real public API with `__get` / `__set` / `__call`. They hide typos, break static analysis, and are slower.
- Do not put business logic in `__destruct`. Shutdown order is not something you want to depend on.

**⚠️ Watch out:**

Do not replace a real public API with `__get` / `__set` / `__call` — typos become “features”, static analysis goes blind. Do not send mail or capture payment from `__destruct`.

**💬 If they follow up:**

- PHP 8.5 `clone($object, [...])`? Wither-style copy for `readonly` classes.
- `__sleep` / `__wakeup`? Soft-deprecated in 8.5; use `__serialize` / `__unserialize`.

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
