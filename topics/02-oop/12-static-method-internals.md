# A static method does not require an object instance to run. How is this managed internally?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Because the method lives on the **class entry**, not on an object.

When PHP loads a class, it stores metadata in a `zend_class_entry`: name, parent, interfaces, property definitions, and a **function table** of methods. A static method is just a function in that table with the `ZEND_ACC_STATIC` flag.

When you call `User::make()`:

1. PHP resolves the class (autoload if needed) — the class is already one shared structure in memory.
2. It looks up `make` in the class function table.
3. It checks the static flag. If you called an instance method statically (in old PHP) you got a warning; in modern PHP you get an Error.
4. It builds a **call frame** (arguments, return slot). It does **not** allocate a `zend_object`.
5. Inside the method, `$this` is not bound. `self::` is the class where the method was **defined**. `static::` is the class that was **called** (late static binding).

```php
class ParentId
{
    public static function who(): string
    {
        return static::class; // late static binding
    }
}

class ChildId extends ParentId {}

ChildId::who(); // "ChildId" — no ChildId object was created
```

That is why static works without `new`: there is nothing to construct. The engine is calling a function that happens to live on the class.

> [!TIP]
> **One-liner:** Static methods are functions on the class metadata (the function table). The engine calls them with a stack frame only — no object is allocated, so there is no `$this`.

**Source:** [PHP: Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — userland rules. Engine view (optional): [PHP Internals Book: object handlers](https://www.phpinternalsbook.com/php7/classes_objects/object_handlers.html) (`get_method` on the class, not an instance; PHP 7+ model, still the basis of 8.x).

**Learn more:**
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `static::class` is the *called* class, with no object
- [Nikita Popov: Internal value representation in PHP 7 (part 2)](https://www.npopov.com/2015/06/19/Internal-value-representation-in-PHP-7-part-2.html) — zvals vs objects in the engine (PHP 7+ model; still the basis of 8.x)
- [Autoloading Classes](https://www.php.net/manual/en/language.oop5.autoload.php) — the class entry is loaded once, then reused for every static call

---

[← Previous](./11-static-vs-instance.md) · [Topic](./README.md) · [Next →](./13-memory-allocation.md)
