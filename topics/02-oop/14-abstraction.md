# What is the concept of Abstraction?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Abstraction means exposing **what** something does and hiding **how** it does it. Callers depend on a small, stable interface; the messy details stay inside.

**Key terms:**

| Term | Plain meaning |
|------|----------------|
| Abstraction | Show what something does; hide how. |
| Abstract class | A class marked `abstract` — a template; you cannot `new` it. |
| Abstract method | A method with a signature but no body — children must write the body. |
| Concrete method | A normal method with a body (already finished). |
| Concrete class | A normal (non-abstract) class you can `new`. |

**Analogy:**

Abstraction is the **checkout button**: you press “pay.” You do not see Stripe JSON, keys, or retries. An `abstract class` is one *language tool* that helps build that idea — not the idea itself.

Two layers people mix up:

- **Abstraction (the idea)** — a `PaymentGateway` with `charge()` so the checkout code never talks to Stripe HTTP APIs.
- **Abstract class (the keyword)** — a language tool that *helps* you build that idea, alongside interfaces.

**Official** ([PHP Manual: Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php)):

```php
abstract class AbstractClass
{
    abstract protected function getValue();

    public function printOut()
    {
        print $this->getValue() . "\n";
    }
}

class ConcreteClass1 extends AbstractClass
{
    protected function getValue()
    {
        return "ConcreteClass1";
    }
}

$class1 = new ConcreteClass1();
$class1->printOut(); // callers use printOut(); they do not know getValue()
```

**In production:**

```php
abstract class PaymentGateway
{
    abstract public function charge(int $cents): string;

    public function receipt(string $id): string
    {
        return "Receipt {$id}";
    }
}

class StripeGateway extends PaymentGateway
{
    public function charge(int $cents): string
    {
        return Http::stripe()->post('/charges', ['amount' => $cents])->json('id');
    }
}

$checkout->pay(new StripeGateway(), $order->totalCents()); // no Stripe JSON in the controller
```

Checkout calls `charge()`. It does not know about API keys, retry policy, or JSON. That is abstraction.

**Watch out:**

Do not mix up **abstraction** (the idea) with **`abstract class`** (one keyword). Interfaces also abstract. Hiding everything behind `__get` is not good abstraction.

**If they follow up:**

- Can you `new` an abstract class? No — next questions.
- Why not only interfaces? When children share real methods/state, an abstract class is the family template.

> [!TIP]
> **One-liner:** Abstraction hides implementation behind a simple public API so callers depend on *what* happens, not *how*.

**Source:** [PHP: Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) — official `abstract class` / `abstract` methods (the language tool behind the idea).

**Learn more:**
- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — abstraction with a contract and no shared code
- [Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — hide `private` details behind a small public API
- [Laravel: Service Container](https://laravel.com/docs/13.x/container) — depending on an abstract type in a real app

---

[← Previous](./13-memory-allocation.md) · [Topic](./README.md) · [Next →](./15-abstract-class-vs-interface.md)
