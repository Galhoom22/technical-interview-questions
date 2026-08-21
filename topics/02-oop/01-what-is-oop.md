# What is OOP?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

OOP (Object-Oriented Programming) is a way of structuring code around **objects**: bundles of **data** (properties) and **behavior** (methods) that represent something in the domain.

Instead of a pile of functions and arrays, you model `User`, `Order`, `Invoice`. Each object knows its own state and the operations that belong to it.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| OOP | Structure code around objects, not around a pile of functions and loose data. |
| Object | A bundle of **data** (properties) and **behavior** (methods) that represents something in the domain. |
| Class | The blueprint. `new ClassName()` creates an object from it. |
| Instance | One object created from a class. Own property values; shared method code. |

### 🧠 Analogy

OOP is modeling the shop as **Order**, **Cart**, **User** — things that *know their data* and *the operations that belong to them* — instead of a pile of functions and loose arrays.

---

### 📘 Official ([PHP Manual: The Basics](https://www.php.net/manual/en/language.oop5.basic.php))

```php
class SimpleClass
{
    public $var = 'a default value';

    public function displayVar()
    {
        echo $this->var;
    }
}

$instance = new SimpleClass();
$instance->displayVar();
```

Data (`$var`) and behavior (`displayVar()`) live on the same object.

### 💼 In production

```php
class Order
{
    public function __construct(private array $itemCents) {}

    public function totalCents(): int
    {
        return array_sum($this->itemCents);
    }
}

$order = new Order([1999, 1299]);
$order->totalCents(); // 3298 — checkout uses the object, not a loose array
```

**Why interviewers ask this:** they want to hear *objects + state + behavior*, not a list of buzzwords. I then connect it to the four principles (encapsulation, abstraction, inheritance, polymorphism) if they follow up.

---

### ⚠️ Watch out

> [!WARNING]
> Listing the four buzzwords with no example is a weak answer. They want *objects + state + behavior*, then the principles if they follow up.

### 💬 If they follow up

> [!NOTE]
> - What are the four principles? Encapsulation, abstraction, inheritance, polymorphism.
> - Why objects instead of functions? The data and the rules that change it live together (`$order->totalCents()`).

---

> [!TIP]
> **One-liner:** OOP models the problem as objects that combine data and the operations on that data, instead of separate functions and unstructured data.

**Source:** [PHP: Classes and Objects](https://www.php.net/manual/en/language.oop5.php) — official OOP chapter this answer is based on.

**Learn more:**
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — `class`, `new`, `$this`, properties, methods
- [PHP: The Right Way](https://phptherightway.com/) — community best-practice guide (not a language spec)
- [PSR-12: Extended Coding Style](https://www.php-fig.org/psr/psr-12/) — the PHP-FIG coding-style PSR
- [PER Coding Style](https://www.php-fig.org/per/coding-style/) — living PHP-FIG style (what Pint’s `per` preset follows)
- [Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — `public` / `protected` / `private` (encapsulation in the language)

---

[Topic](./README.md) · [Next →](./02-what-is-a-class.md)