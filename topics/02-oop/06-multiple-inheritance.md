# How do you implement multiple inheritance in PHP?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

You don’t — **PHP allows a class to extend only one parent**. Multiple inheritance of classes is not supported.

What you do instead:

| Need | Tool |
|------|------|
| Multiple *contracts* | `implements InterfaceA, InterfaceB` |
| Reuse *implementation* across unrelated classes | `use TraitA, TraitB` |
| Reuse *implementation* in a family of types | Single inheritance (`extends`) |
| Reuse by *delegation* | Composition — inject another object |

```php
interface Auditable
{
    public function auditId(): string;
}

interface Cacheable
{
    public function cacheKey(): string;
}

class Report extends BaseReport implements Auditable, Cacheable
{
    use LogsActivity; // shared implementation

    public function auditId(): string { return (string) $this->id; }
    public function cacheKey(): string { return 'report:'.$this->id; }
}
```

If an interviewer says “so traits are multiple inheritance?”, I correct it: traits are compiler-assisted copy-paste, not a second parent class. There is no diamond-problem of *types* — only method-name conflicts you must resolve explicitly.

> [!TIP]
> **One-liner:** PHP has no multiple class inheritance. I combine one `extends`, many `implements`, traits for shared code, and composition when behavior belongs to another object.

**Source:** [PHP: Traits](https://www.php.net/manual/en/language.oop5.traits.php) — PHP’s documented substitute for multiple inheritance of *implementation*; plus [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) (single `extends` only).

**Learn more:**
- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — multiple *contracts* via `implements A, B`
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — composition: hold another object as a property instead of extending it
- [Laravel: Service Container](https://laravel.com/docs/13.x/container) — injecting collaborators instead of inheriting them

---

[← Previous](./05-trait.md) · [Topic](./README.md) · [Next →](./07-destructor.md)
