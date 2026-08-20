# How do you instantiate an object from a class? Explain the step-by-step process of how the object is built and what handles its creation.

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

From user code it looks like one step: `new ClassName(...)`. Internally it is a pipeline the **Zend Engine** runs.

```php
$user = new User('Omar', 'omar@example.com');
```

**Step by step:**

1. **Load the class** — if `User` is not in memory, the autoloader (Composer PSR-4) includes the file. If that fails: “class not found”.
2. **`new` runs** — the engine executes the `NEW` opcode. This is not your constructor yet.
3. **Allocate the object** — Zend creates a `zend_object` in the object store: header, class pointer, property slots, GC info. Memory is on the heap; PHP hands you a handle (the zval points at that object).
4. **Initialize properties** — default values are applied. Typed properties with no default stay **uninitialized** until assigned (reading them throws).
5. **Run `__construct`** — if it exists, it is called with the arguments you passed. This is *your* code, after the object already exists.
6. **Return the instance** — the constructor’s return value is ignored. The result of `new` is the object from step 3.

If `__construct` throws, the object never becomes a usable variable; it will be destroyed.

In Laravel, `app(UserService::class)` is not `new` in your controller — the **Service Container** resolves constructor dependencies, then it still ends up doing the same `new` pipeline.

> [!TIP]
> **One-liner:** `new` is handled by the Zend Engine: autoload the class, allocate the object, init properties, then call `__construct`. The constructor does not create the object — it initializes an object the engine already built.

**Source:** [PHP: The Basics](https://www.php.net/manual/en/language.oop5.basic.php) (`new`) plus [Constructors](https://www.php.net/manual/en/language.oop5.decon.php) — constructor runs *after* the object exists.

**Learn more:**
- [Autoloading Classes](https://www.php.net/manual/en/language.oop5.autoload.php) — step 1: load the class file
- [PSR-4: Autoloader](https://www.php-fig.org/psr/psr-4/) — how Composer maps `App\User` to a path
- [Nikita Popov: Internal value representation in PHP 7 (part 2)](https://www.npopov.com/2015/06/19/Internal-value-representation-in-PHP-7-part-2.html) — objects, handles, and heap layout (PHP 7+ engine model; still the basis of 8.x)
- [PHP Internals Book: object handlers](https://www.phpinternalsbook.com/php7/classes_objects/object_handlers.html) — `get_constructor` and object creation hooks (community internals book, not the PHP Manual)

---

[← Previous](./08-private-constructor.md) · [Topic](./README.md) · [Next →](./10-instantiate-with-private-constructor.md)
