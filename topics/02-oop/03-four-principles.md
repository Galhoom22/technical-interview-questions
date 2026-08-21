# What are the four core principles of OOP?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

The four names interviewers want, then how they show up in PHP:

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Encapsulation | Hide internal state; expose a small public API. |
| Abstraction | Show what something does; hide how. |
| Inheritance | A child reuses / specializes a parent (`extends`). |
| Polymorphism | Same message, different behavior. |
| Composition | Hold another object as a property and delegate to it, instead of extending it. |

### 🧠 Analogy

A vending machine **hides** its coins (encapsulation), you only press “coffee” (abstraction), a specialty machine can **reuse** the base box (inheritance), and “brew” means espresso in one machine and filter in another (polymorphism). Prefer plugging in a grinder (composition) over a family tree of machines.

| Principle | In practice |
|-----------|-------------|
| **Encapsulation** | `private` properties, public methods |
| **Abstraction** | Interfaces, abstract classes |
| **Inheritance** | `class Admin extends User` |
| **Polymorphism** | `Payable[]` of `Invoice` and `Salary` |

---

### 📘 Official ([PHP Manual: Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php))

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

### 💼 In production

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

---

> [!WARNING]
> Inheritance is the principle people overuse. Deep `extends` trees hurt more than they help. Lead with encapsulation and polymorphism; mention inheritance last.

> [!NOTE]
> - Composition vs inheritance? I inject a collaborator instead of extending, unless it really *is* a kind of the parent.
> - Where is polymorphism in Laravel? Any interface bound in the container (`Mailer` → `SmtpMailer`).

---

> [!TIP]
> **One-liner:** Encapsulation hides state, abstraction hides complexity, inheritance shares structure, polymorphism lets different classes answer the same message.

**Source:** [PHP: Classes and Objects](https://www.php.net/manual/en/language.oop5.php) — the four principles map onto these language features (visibility, abstract/interface, `extends`, interfaces).

**Learn more:**
- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — `extends`, overriding, `parent::`
- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — polymorphism via a shared contract
- [Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) — abstract classes as a shared family

---

[← Previous](./02-what-is-a-class.md) · [Topic](./README.md) · [Next →](./04-object-instantiation.md)