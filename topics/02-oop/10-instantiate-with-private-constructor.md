# If a constructor is private, how can you instantiate an object of that class?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

You instantiate it **from inside the class**, usually with a **static factory** (or `getInstance()` for a singleton). `new self(...)` / `new static(...)` is allowed there because it is the same class.

```php
class Database
{
    private static ?self $instance = null;

    private function __construct(private string $dsn) {}

    public static function getInstance(): self
    {
        return self::$instance ??= new self(config('database.dsn'));
    }
}

Database::getInstance();
```

Other legitimate paths:

- A **public static factory**: `Order::fromCart($cart)` → `return new self(...)`.
- A **friend class in the same file** cannot bypass it — visibility is per class, not per file.
- **Reflection** (`ReflectionClass::newInstanceWithoutConstructor()`) can cheat. That is for frameworks/tests, not production domain code.

A **child class** cannot call a **private** parent constructor. If subclasses should exist, the constructor should be `protected`.

> [!TIP]
> **One-liner:** Call a public static method on the class; that method is allowed to `new self()`. Outside code never uses `new` directly.

**Source:** [PHP: Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — `private` constructor is visible only inside the same class, so a public static factory can still `new self()`.

**Learn more:**
- [Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — factory methods without an instance
- [Constructors and Destructors](https://www.php.net/manual/en/language.oop5.decon.php) — `protected` constructor if subclasses should exist
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `new static()` vs `new self()` in factories

---

[← Previous](./09-object-instantiation.md) · [Topic](./README.md) · [Next →](./11-static-vs-instance.md)
