# Can a constructor be private?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Yes. A `private` constructor means **`new ClassName()` is only legal inside that class**. Code outside cannot instantiate it directly. `protected` allows children, but still blocks the outside world.

```php
class Uuid
{
    private function __construct(private string $value) {}

    public static function fromString(string $value): self
    {
        // validate, then:
        return new self($value);
    }
}

Uuid::fromString('…'); // ok
new Uuid('…');         // Error: private constructor
```

**Why you do this:**

- Named constructors / factories (`fromString`, `fromArray`) that validate first
- Singleton (I mention it because they ask; I rarely use it)
- Forcing all creation through a factory or the container

> [!TIP]
> **One-liner:** Yes. Private constructors block `new` from outside and force creation through a static factory (or similar) inside the class.

**Source:** [PHP: Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — methods (including `__construct`) can be `private` / `protected`; plus [Constructors](https://www.php.net/manual/en/language.oop5.decon.php).

**Learn more:**
- [Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — named constructors / factories as `public static function fromString()`
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — `new self()` / `new static()` from inside the class
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `new static()` in a parent factory used by a child

---

[← Previous](./07-destructor.md) · [Topic](./README.md) · [Next →](./09-object-instantiation.md)
