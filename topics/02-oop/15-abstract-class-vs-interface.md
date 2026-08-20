# What is the difference between an Abstract Class and an Interface?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Both define a contract. An **abstract class** can also ship shared code and state. An **interface** is only a contract (multiple allowed).

| Aspect | Abstract class | Interface |
|--------|----------------|-----------|
| Instantiated? | No | No |
| Methods | Abstract + concrete | Signatures (no body, historically) |
| Properties / constructor | Yes | No properties historically; PHP 8.4+ (including **8.5**) adds property hooks on interfaces |
| Visibility | public / protected / private | Public methods |
| Inheritance | A class `extends` **one** | A class `implements` **many** |
| Constants | Yes | Yes |
| Typical role | Shared base in one family | Capability / role (`Payable`, `Jsonable`) |

```php
abstract class Report
{
    public function __construct(protected string $title) {}

    abstract public function generate(): string;

    public function heading(): string
    {
        return strtoupper($this->title);
    }
}

interface Exportable
{
    public function export(): string;
}

class PdfReport extends Report implements Exportable
{
    public function generate(): string { return '%PDF…'; }
    public function export(): string { return $this->generate(); }
}
```

**How I choose:** if types only share a *role*, I use an interface. If they share a *family* and real code/state, I use an abstract class. Often both: abstract base + interface for the public contract.

> [!TIP]
> **One-liner:** An interface is a pure contract (a class can have many). An abstract class is a partial implementation you extend once — it can hold properties and real methods.

**Source:** [PHP: Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) and [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — official side-by-side rules this table is based on.

**Learn more:**
- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — why you get only one parent class
- [PHP 8.5 new features](https://www.php.net/manual/en/migration85.new-features.php) — current language; interface property hooks (8.4+) still apply
- [Traits](https://www.php.net/manual/en/language.oop5.traits.php) — a third option when you want reuse without an abstract parent

---

[← Previous](./14-abstraction.md) · [Topic](./README.md) · [Next →](./16-instantiate-abstract-class.md)
