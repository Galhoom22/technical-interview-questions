# What is a Class?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

A class is the **blueprint**. An object (instance) is a **value built from that blueprint**.

The class defines:

- properties (state)
- methods (behavior)
- visibility (`public` / `protected` / `private`)
- how instances are constructed

**Official** ([PHP Manual: The Basics](https://www.php.net/manual/en/language.oop5.basic.php)):

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
```

The class is the declaration. `$instance` is one object built from it.

**In production:**

```php
class User
{
    public function __construct(
        public string $name,
        public string $email,
    ) {}

    public function mention(): string
    {
        return '@'.$this->name;
    }
}

$omar = new User('Omar', 'omar@shop.example');
$sara = new User('Sara', 'sara@shop.example'); // same class, two accounts
```

One class, many objects. Each instance has its own property values; they share the method code.

> [!TIP]
> **One-liner:** A class is the template; `new ClassName()` creates an object from that template.

**Source:** [PHP: The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — official `class` / `new` / instance page this answer follows.

**Learn more:**
- [Constructors and Destructors](https://www.php.net/manual/en/language.oop5.decon.php) — how the object is initialized after `new`
- [Properties](https://www.php.net/manual/en/language.oop5.properties.php) — typed properties, `readonly`, defaults
- [Autoloading Classes](https://www.php.net/manual/en/language.oop5.autoload.php) — the class file must load before `new` works

---

[← Previous](./02-four-principles.md) · [Topic](./README.md) · [Next →](./04-interface.md)
