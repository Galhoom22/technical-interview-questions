# If a constructor is private, how can you instantiate an object of that class?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

You instantiate it **from inside the class**, usually with a **static factory** (or `getInstance()` for a singleton). `new self(...)` / `new static(...)` is allowed there because it is the same class.

**🔑 Key terms:**

| Term | Plain meaning |
|------|----------------|
| Private constructor | `__construct` only the same class can call. Blocks `new` from outside. |
| Static factory | A public static method that returns `new self()` / `new static()`. |
| `new self()` | Create an instance of the class where this code is **written**. |
| `new static()` | Create an instance of the class that was **called** (late static binding). |
| Reflection | Can bypass a constructor (`newInstanceWithoutConstructor`). Framework/tests — not domain code. |

**🧠 Analogy:**

The locked door still opens from **inside**. A public static window (`fromCart`) is allowed to `new self()`. Reflection picking the lock is for the framework, not checkout.

**📘 Official** ([PHP Manual: Static Keyword](https://www.php.net/manual/en/language.oop5.static.php)):

```php
class Foo
{
    public static function aStaticMethod()
    {
        // no $this — this is a function on the class
    }
}

Foo::aStaticMethod();
$classname = 'Foo';
$classname::aStaticMethod();
```

**💼 In production:**

```php
class Order
{
    private function __construct(private array $lines) {}

    public static function fromCart(Cart $cart): self
    {
        return new self($cart->lines()); // new self is allowed here
    }
}

Order::fromCart($cart);
new Order([]); // blocked if the constructor is private
```

Other legitimate paths:

- A **public static factory**: `Order::fromCart($cart)` → `return new self(...)`.
- A **friend class in the same file** cannot bypass it — visibility is per class, not per file.
- **Reflection** (`ReflectionClass::newInstanceWithoutConstructor()`) can cheat. That is for frameworks/tests, not production domain code.

A **child class** cannot call a **private** parent constructor. If subclasses should exist, the constructor should be `protected`.

**⚠️ Watch out:**

Visibility is per class, not per file. A “friend” in the same file cannot `new` it. Reflection can cheat — say that out loud as a test/framework trick, not as the design.

**💬 If they follow up:**

- `new self()` vs `new static()`? `self` is the class where the code is written; `static` is the class that was called.
- Singleton? I mention `getInstance()` because they ask; I rarely use it in Laravel (the container already holds one).

> [!TIP]
> **One-liner:** Call a public static method on the class; that method is allowed to `new self()`. Outside code never uses `new` directly.

**Source:** [PHP: Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — `private` constructor is visible only inside the same class, so a public static factory can still `new self()`.

**Learn more:**
- [Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — factory methods without an instance
- [Constructors and Destructors](https://www.php.net/manual/en/language.oop5.decon.php) — `protected` constructor if subclasses should exist
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `new static()` vs `new self()` in factories

---

[← Previous](./09-object-instantiation.md) · [Topic](./README.md) · [Next →](./11-static-vs-instance.md)
