# What is the difference between static and non-static methods?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

An **instance** method runs on an object (`$this`). A **static** method runs on the class (no `$this`).

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Instance method | Belongs to an object. Uses `$this`. Call with `$obj->method()`. |
| Static method | Belongs to the class. No `$this`. Call with `ClassName::method()`. |
| `$this` | The current object. Not available in a static method. |
| Static property | Data that lives once on the class, not per object. |

### 🧠 Analogy

Instance methods are **this cart’s** total. Static methods are the **store policy** on the wall — no cart needed (`Cart::empty()`).

| Aspect | Instance (non-static) | Static |
|--------|----------------------|--------|
| Belongs to | An object | The class |
| Call | `$user->name()` | `User::name()` or `static::name()` |
| `$this` | Yes — current instance | No — using `$this` throws |
| State | Instance properties | Only static properties / arguments |
| Typical use | Behavior that needs this object’s data | Factories, utilities, named constructors |

---

### 📘 Official ([PHP Manual: Static Keyword](https://www.php.net/manual/en/language.oop5.static.php))

```php
class Foo
{
    public static $my_static = 'foo';

    public function staticValue()
    {
        return self::$my_static; // instance method may read static data
    }
}

print Foo::$my_static;
$foo = new Foo();
print $foo->staticValue();
```

### 💼 In production

```php
class Cart
{
    public function __construct(private array $lines) {}

    public function totalCents(): int
    {
        return array_sum(array_column($this->lines, 'cents')); // needs this cart
    }

    public static function empty(): self
    {
        return new self([]); // named constructor — no existing cart
    }
}

Cart::empty();
$cart->totalCents();
```

Static methods are not “faster OOP”. They are functions namespaced on a class. Too many of them usually means the design wanted a service object instead.

---

### ⚠️ Watch out

> [!WARNING]
> `$this` inside a static method throws. Static is not a performance trick. A class full of static utilities usually wanted a service object.

### 💬 If they follow up

> [!NOTE]
> - Can an instance method read static data? Yes (`self::$count`). The reverse cannot use instance state.
> - Late static binding? `static::` is the *called* class — see the next question.

---

> [!TIP]
> **One-liner:** Instance methods run on an object and use `$this`. Static methods run on the class, have no `$this`, and cannot use instance state.

**Source:** [PHP: Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — official rules for static methods, static properties, and “no `$this`”.

**Learn more:**
- [Paamayim Nekudotayim (`::`)](https://www.php.net/manual/en/language.oop5.paamayim-nekudotayim.php) — `ClassName::method()` vs `$obj->method()`
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `self::` vs `static::`
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — `$this` on instance methods

---

[← Previous](./10-instantiate-with-private-constructor.md) · [Topic](./README.md) · [Next →](./12-static-method-internals.md)
