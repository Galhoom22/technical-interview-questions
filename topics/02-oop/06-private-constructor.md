# Can a constructor be private?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Yes. A `private` constructor means **`new ClassName()` is only legal inside that class**. Code outside cannot instantiate it directly. `protected` allows children, but still blocks the outside world.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Private constructor | `__construct` only the same class can call. Blocks `new` from outside. |
| Visibility | `public` / `protected` / `private` — who can access a member. |
| Static factory | A public static method that returns `new self()` / `new static()`. |
| Named constructor | A factory with a clear name (`fromString`, `fromCart`) instead of a raw `new`. |

### 🧠 Analogy

A private constructor is a **locked factory door**. Customers cannot walk in and `new` junk. They use the public window (`fromString`) so only valid IDs get made.

---

### 📘 Official ([PHP Manual: Visibility](https://www.php.net/manual/en/language.oop5.visibility.php))

```php
class MyClass
{
    private function __construct() {}

    public static function from()
    {
        return new self(); // legal: same class
    }
}

MyClass::from();
new MyClass(); // Error: private constructor
```

Methods, including `__construct`, can be `private` / `protected` / `public`.

### 💼 In production

```php
class OrderId
{
    private function __construct(private string $uuid) {}

    public static function fromString(string $value): self
    {
        if (! str_is_uuid($value)) {
            throw new InvalidArgumentException('Invalid order id');
        }

        return new self($value); // only valid UUIDs become objects
    }
}

OrderId::fromString($request->string('order_id'));
new OrderId('nope'); // not allowed — keeps junk IDs out of the domain
```

**Why you do this:**

- Named constructors / factories (`fromString`, `fromArray`) that validate first
- Singleton (I mention it because they ask; I rarely use it)
- Forcing all creation through a factory or the container

---

> [!WARNING]
> `protected` still allows children to `new`. `private` does not. Do not reach for a Singleton unless they ask — named factories are the usual real answer.

> [!NOTE]
> - How do you instantiate then? Public static factory → `new self()` inside the class.
> - Can a child call it? Not if it is `private`. Use `protected` if subclasses should exist.

---

> [!TIP]
> **One-liner:** Yes. Private constructors block `new` from outside and force creation through a static factory (or similar) inside the class.

**Source:** [PHP: Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — methods (including `__construct`) can be `private` / `protected`; plus [Constructors](https://www.php.net/manual/en/language.oop5.decon.php).

**Learn more:**
- [Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — named constructors / factories as `public static function fromString()`
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — `new self()` / `new static()` from inside the class
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `new static()` in a parent factory used by a child

---

[← Previous](./05-destructor.md) · [Topic](./README.md) · [Next →](./07-instantiate-with-private-constructor.md)