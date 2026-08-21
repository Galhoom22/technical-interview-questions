# Can you instantiate an object directly from an Abstract Class?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

No. An abstract class is incomplete by definition. PHP throws an error if you `new` it.

You instantiate a **concrete child** that implements every abstract method.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Abstract class | A class marked `abstract` — a template; you cannot `new` it. |
| Abstract method | A method with a signature but no body — children must write the body. |
| Concrete class | A normal (non-abstract) class you can `new`. |
| Instantiate | Create an object (`new`, or a factory that calls `new` inside). |
| Type-hint | Require a type (`Discount $d`). You can hint the abstract type and pass a child. |

### 🧠 Analogy

An abstract class is a **blank government form**. You cannot submit the empty template (`new Discount`). Each department fills a completed copy (`CouponDiscount`). You can still *ask* for “any discount form” (type-hint) and receive a filled one.

---

### 📘 Official ([PHP Manual: Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php))

```php
abstract class AbstractClass
{
    abstract protected function getValue();
}

// $obj = new AbstractClass(); // Error: Cannot instantiate abstract class

class ConcreteClass1 extends AbstractClass
{
    protected function getValue()
    {
        return "ConcreteClass1";
    }
}

$obj = new ConcreteClass1(); // ok
```

### 💼 In production

```php
abstract class Discount
{
    abstract public function apply(int $cents): int;
}

class CouponDiscount extends Discount
{
    public function apply(int $cents): int
    {
        return (int) round($cents * 0.9);
    }
}

new Discount();        // Error — there is no generic discount
new CouponDiscount();  // checkout uses a real rule

function quote(Discount $discount, int $cents): int
{
    return $discount->apply($cents); // type-hint abstract, pass a child
}
```

---

### ⚠️ Watch out

> [!WARNING]
> If the child skips an abstract method, PHP still treats it as incomplete (error / still abstract). Forgetting `abstract` on the class when it has abstract methods is also fatal.

### 💬 If they follow up

> [!NOTE]
> - Can you type-hint the abstract class? Yes — pass a concrete child.
> - Same for interfaces? Yes — you never `new` an interface either.

---

> [!TIP]
> **One-liner:** No. Abstract classes cannot be instantiated. You `new` a concrete subclass that fills in the abstract methods.

**Source:** [PHP: Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) — “PHP … will issue an error if you try to instantiate an abstract class.”

**Learn more:**
- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — the concrete child you *can* `new`
- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — same rule: you never `new` an interface
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — type-hint the abstract type, pass a child instance

---

[← Previous](./15-abstract-class-vs-interface.md) · [Topic](./README.md)
