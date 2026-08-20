# What is a Destructor in PHP, and what is its purpose?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

A destructor is `__destruct()`. PHP calls it when the object is **destroyed**: no more references to it, or the request is shutting down.

Its purpose is **cleanup** of resources the garbage collector will not close for you in time: file handles, some native connections, temporary files.

```php
class FileBuffer
{
    private $handle;

    public function __construct(string $path)
    {
        $this->handle = fopen($path, 'a');
    }

    public function __destruct()
    {
        if (is_resource($this->handle)) {
            fclose($this->handle);
        }
    }
}
```

**Practical points for the interview:**

- I do **not** put business logic here (sending emails, writing orders). Destruction order during shutdown is not a workflow engine.
- Prefer explicit `close()` / `try/finally` / wrapping in a class that is deterministic.
- In typical Laravel apps I almost never write `__destruct`. The framework and PHP close PDO, files, and streams at the end of the request.

> [!TIP]
> **One-liner:** `__destruct` runs when the object is destroyed. Use it for resource cleanup, not for business logic.

**Source:** [PHP: Constructors and Destructors](https://www.php.net/manual/en/language.oop5.decon.php) — official `__destruct` timing and purpose.

**Learn more:**
- [Garbage Collection](https://www.php.net/manual/en/features.gc.php) — refcounting vs cycle collector (why destructors are not a workflow engine)
- [PHP Internals Book: object handlers](https://www.phpinternalsbook.com/php7/classes_objects/object_handlers.html) — `dtor_obj` vs `free_obj` (engine view of destruction; PHP 7+ model, still the basis of 8.x)
- [Magic Methods](https://www.php.net/manual/en/language.oop5.magic.php) — where `__destruct` sits among the other hooks

---

[← Previous](./06-multiple-inheritance.md) · [Topic](./README.md) · [Next →](./08-private-constructor.md)
