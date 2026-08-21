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

**Official** ([PHP Manual: Traits](https://www.php.net/manual/en/language.oop5.traits.php) — “an alternative to multiple inheritance”):

```php
class Base {
    public function sayHello() {
        echo 'Hello ';
    }
}

trait SayWorld {
    public function sayHello() {
        parent::sayHello();
        echo 'World!';
    }
}

class MyHelloWorld extends Base {
    use SayWorld;
}

$o = new MyHelloWorld();
$o->sayHello(); // Hello World!
```

One `extends`, extra behavior via `use`. A class still cannot `extend A, B`.

**In production:**

```php
interface Auditable { public function auditId(): string; }
interface Billable { public function billableAmount(): int; }

class Order extends Model implements Auditable, Billable
{
    use HasFactory; // Laravel trait — shared code, not a second parent

    public function auditId(): string { return 'order:'.$this->id; }
    public function billableAmount(): int { return $this->total_cents; }
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
