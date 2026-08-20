# Can you instantiate an object directly from an Abstract Class?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

No. An abstract class is incomplete by definition. PHP throws an error if you `new` it.

You instantiate a **concrete child** that implements every abstract method.

```php
abstract class Animal
{
    abstract public function speak(): string;
}

class Dog extends Animal
{
    public function speak(): string
    {
        return 'woof';
    }
}

new Animal(); // Error: Cannot instantiate abstract class Animal
new Dog();    // ok
```

You *can* type-hint the abstract class and receive a child. That is polymorphism, not instantiation of the abstract type.

```php
function handle(Animal $animal): string
{
    return $animal->speak();
}

handle(new Dog());
```

> [!TIP]
> **One-liner:** No. Abstract classes cannot be instantiated. You `new` a concrete subclass that fills in the abstract methods.

**Source:** [PHP: Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) — “PHP … will issue an error if you try to instantiate an abstract class.”

**Learn more:**
- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — the concrete child you *can* `new`
- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — same rule: you never `new` an interface
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — type-hint the abstract type, pass a child instance

---

[← Previous](./15-abstract-class-vs-interface.md) · [Topic](./README.md)
