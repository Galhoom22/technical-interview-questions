# What is the difference between an Abstract Class and an Interface?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Both define a contract. An **abstract class** can also ship shared code and state. An **interface** is only a contract (multiple allowed).

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Abstract class | A class marked `abstract` — a template; you cannot `new` it. |
| Abstract method | A method with a signature but no body — children must write the body. |
| Concrete method | A normal method with a body (already finished). |
| Interface | A contract of public methods a class must implement. It does not say *how*. |
| Concrete class | A normal (non-abstract) class you can `new`. |

### 🧠 Analogy

An interface is a **job contract** (many allowed). An abstract class is a **half-finished kit** — some parts already built, some blanks the child must fill — and you get only one kit (`extends`).

| Aspect | Abstract class | Interface |
|--------|----------------|-----------|
| Instantiated? | No | No |
| Methods | Abstract + concrete | Signatures (no body, historically) |
| Properties / constructor | Yes | No properties historically; PHP 8.4+ (including **8.5**) adds property hooks on interfaces |
| Visibility | public / protected / private | Public methods |
| Inheritance | A class `extends` **one** | A class `implements` **many** |
| Constants | Yes | Yes |
| Typical role | Shared base in one family | Capability / role (`Payable`, `Jsonable`) |

---

### 📘 Official ([PHP Manual: Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) + [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php))

```php
abstract class AbstractClass
{
    abstract protected function getValue();

    public function printOut()
    {
        print $this->getValue();
    }
}

interface Template
{
    public function setVariable($name, $var);
    public function getHtml($template);
}
```

Abstract class: shared `printOut()`. Interface: contract only — a class may `implement` several.

### 💼 In production

```php
abstract class Report
{
    public function __construct(protected Order $order) {}

    abstract public function generate(): string;

    public function filename(): string
    {
        return 'order-'.$this->order->id;
    }
}

interface Downloadable
{
    public function downloadHeaders(): array;
}

class InvoicePdf extends Report implements Downloadable
{
    public function generate(): string { /* TCPDF / Dompdf */ return '%PDF'; }
    public function downloadHeaders(): array { return ['Content-Type' => 'application/pdf']; }
}
```

**How I choose:** if types only share a *role*, I use an interface. If they share a *family* and real code/state, I use an abstract class. Often both: abstract base + interface for the public contract.

---

> [!WARNING]
> You cannot `new` either. PHP 8.4+ property hooks on interfaces exist — do not recite “interfaces never have properties” as if it were still absolute.

> [!NOTE]
> - Can a class have both? Yes: `extends Report implements Downloadable`.
> - Multiple abstract classes? No — one `extends`.

---

> [!TIP]
> **One-liner:** An interface is a pure contract (a class can have many). An abstract class is a partial implementation you extend once — it can hold properties and real methods.

**Source:** [PHP: Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) and [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — official side-by-side rules this table is based on.

**Learn more:**
- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — why you get only one parent class
- [PHP 8.5 new features](https://www.php.net/manual/en/migration85.new-features.php) — current language; interface property hooks (8.4+) still apply
- [Traits](https://www.php.net/manual/en/language.oop5.traits.php) — a third option when you want reuse without an abstract parent

---

[← Previous](./14-abstraction.md) · [Topic](./README.md) · [Next →](./16-instantiate-abstract-class.md)