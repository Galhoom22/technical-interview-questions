# What is a Trait?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

A trait is a reuse tool for **horizontal** sharing of methods (and properties). PHP copies the trait’s members into the class at compile time. It is not inheritance and not an interface.

**Official** ([PHP Manual: Traits](https://www.php.net/manual/en/language.oop5.traits.php)):

```php
trait ezcReflectionReturnInfo {
    function getReturnType() { /*1*/ }
    function getReturnDescription() { /*2*/ }
}

class ezcReflectionMethod extends ReflectionMethod {
    use ezcReflectionReturnInfo;
}

class ezcReflectionFunction extends ReflectionFunction {
    use ezcReflectionReturnInfo;
}
```

Two unrelated classes reuse the same methods. Conflict resolution uses `insteadof` / `as`.

**In production:**

```php
trait HasSlug
{
    public function slug(string $value): string
    {
        return $value
            |> (fn (string $s): string => str_replace(' ', '-', $s))
            |> strtolower(...);
    }
}

class Product
{
    use HasSlug; // catalog URLs
}

class Article
{
    use HasSlug; // blog URLs — no shared parent with Product
}

(new Product())->slug('Blue T-Shirt'); // "blue-t-shirt"
```

**When I use traits:** small, focused behavior used in several classes that do **not** share a parent (`HasFactory` in Laravel is the classic example).

**When I avoid them:** as a dumping ground for unrelated methods. If the trait needs a lot of hidden state, it probably wants to be a collaborator object instead (composition).

> [!TIP]
> **One-liner:** A trait is copy-paste reuse managed by the language — a way to share methods across classes without multiple inheritance.

**Source:** [PHP: Traits](https://www.php.net/manual/en/language.oop5.traits.php) — official `use`, conflict resolution (`insteadof` / `as`), and the “not inheritance” wording.

**Learn more:**
- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — what traits are *not* (a second parent class)
- [Laravel Eloquent: factories](https://laravel.com/docs/13.x/eloquent-factories) — `HasFactory` is the trait you already use in Laravel
- [Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — how trait members become part of the class

---

[← Previous](./04-interface.md) · [Topic](./README.md) · [Next →](./06-multiple-inheritance.md)
