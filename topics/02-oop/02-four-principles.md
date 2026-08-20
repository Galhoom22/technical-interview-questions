# What are the four core principles of OOP?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

| Principle | Meaning | In practice |
|-----------|---------|-------------|
| **Encapsulation** | Hide internal state; expose a small public API | `private` properties, public methods |
| **Abstraction** | Show what something does, hide how | Interfaces, abstract classes |
| **Inheritance** | A child reuses / specializes a parent | `class Admin extends User` |
| **Polymorphism** | Same message, different behavior | `Payable[]` of `Invoice` and `Salary` |

```php
interface Payable
{
    public function pay(): void;
}

class Invoice implements Payable
{
    public function pay(): void { /* charge customer */ }
}

class Salary implements Payable
{
    public function pay(): void { /* pay employee */ }
}

function settle(Payable $item): void
{
    $item->pay(); // same call, different implementation
}
```

I mention inheritance last on purpose. In PHP I prefer **composition + interfaces** over deep class trees. Inheritance is a tool, not the goal.

> [!TIP]
> **One-liner:** Encapsulation hides state, abstraction hides complexity, inheritance shares structure, polymorphism lets different classes answer the same message.

**Source:** [PHP: Classes and Objects](https://www.php.net/manual/en/language.oop5.php) — the four principles map onto these language features (visibility, abstract/interface, `extends`, interfaces).

**Learn more:**
- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — `extends`, overriding, `parent::`
- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — polymorphism via a shared contract
- [Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) — abstract classes as a shared family

---

[← Previous](./01-what-is-oop.md) · [Topic](./README.md) · [Next →](./03-what-is-a-class.md)
