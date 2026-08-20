# What is OOP?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

OOP (Object-Oriented Programming) is a way of structuring code around **objects**: bundles of **data** (properties) and **behavior** (methods) that represent something in the domain.

Instead of a pile of functions and arrays, you model `User`, `Order`, `Invoice`. Each object knows its own state and the operations that belong to it.

```php
class Order
{
    public function __construct(
        private array $items,
    ) {}

    public function total(): int
    {
        return array_sum($this->items);
    }
}

$order = new Order([1000, 2500]);
$order->total(); // 3500
```

**Why interviewers ask this:** they want to hear *objects + state + behavior*, not a list of buzzwords. I then connect it to the four principles (encapsulation, abstraction, inheritance, polymorphism) if they follow up.

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

[Topic](./README.md) · [Next →](./02-four-principles.md)
