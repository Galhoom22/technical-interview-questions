# What is the difference between static and non-static methods?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

| Aspect | Instance (non-static) | Static |
|--------|----------------------|--------|
| Belongs to | An object | The class |
| Call | `$user->name()` | `User::name()` or `static::name()` |
| `$this` | Yes — current instance | No — using `$this` throws |
| State | Instance properties | Only static properties / arguments |
| Typical use | Behavior that needs this object’s data | Factories, utilities, named constructors |

```php
class Counter
{
    private int $value = 0;
    private static int $created = 0;

    public function __construct()
    {
        self::$created++;
    }

    public function increment(): void   // needs $this
    {
        $this->value++;
    }

    public static function created(): int  // no instance needed
    {
        return self::$created;
    }
}

$c = new Counter();
$c->increment();
Counter::created();
```

Static methods are not “faster OOP”. They are functions namespaced on a class. Too many of them usually means the design wanted a service object instead.

> [!TIP]
> **One-liner:** Instance methods run on an object and use `$this`. Static methods run on the class, have no `$this`, and cannot use instance state.

**Source:** [PHP: Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — official rules for static methods, static properties, and “no `$this`”.

**Learn more:**
- [Paamayim Nekudotayim (`::`)](https://www.php.net/manual/en/language.oop5.paamayim-nekudotayim.php) — `ClassName::method()` vs `$obj->method()`
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `self::` vs `static::`
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — `$this` on instance methods

---

[← Previous](./10-instantiate-with-private-constructor.md) · [Topic](./README.md) · [Next →](./12-static-method-internals.md)
