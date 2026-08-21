# What are the four core principles of OOP?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

| Principle | Meaning | In practice |
|-----------|---------|-------------|
| **Encapsulation** | Hide internal state; expose a small public API | `private` properties, public methods |
| **Abstraction** | Show what something does, hide how | Interfaces, abstract classes |
| **Inheritance** | A child reuses / specializes a parent | `class Admin extends User` |
| **Polymorphism** | Same message, different behavior | `Payable[]` of `Invoice` and `Salary` |

**Official** ([PHP Manual: Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php)):

```php
interface Template
{
    public function setVariable($name, $var);
    public function getHtml($template);
}

class WorkingTemplate implements Template
{
    private $vars = [];

    public function setVariable($name, $var)
    {
        $this->vars[$name] = $var;
    }

    public function getHtml($template)
    {
        foreach ($this->vars as $name => $value) {
            $template = str_replace('{'.$name.'}', $value, $template);
        }

        return $template;
    }
}
```

Same `Template` type, different classes can implement it (polymorphism). Visibility on properties is encapsulation.

**In production:**

```php
interface Payable
{
    public function charge(int $cents): string;
}

class StripeGateway implements Payable
{
    public function charge(int $cents): string { /* Stripe HTTP */ return 'ch_123'; }
}

class CodGateway implements Payable
{
    public function charge(int $cents): string { return 'cod'; }
}

function checkout(Payable $gateway, int $cents): string
{
    return $gateway->charge($cents); // checkout does not care which provider
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
