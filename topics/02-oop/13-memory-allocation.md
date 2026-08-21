# What is the difference in memory (RAM) allocation between instantiating an object and calling a static method?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

`new` pays for a **heap object** every time. A static call pays for the **already-loaded class** plus a short-lived stack frame.

**Key terms:**

| Term | Plain meaning |
|------|----------------|
| Heap | Where each object lives until nothing references it. |
| Stack frame | Memory for one call (arguments, locals). Popped when the method returns. |
| Class metadata | Loaded **once** (the class entry). Shared by every `new` and every static call. |
| Instance | One object created from a class. Own property values; shared method code. |

**Analogy:**

`new` is buying a **new chair** (heap, stays until thrown away). A static call is **reading the fire-exit sign** already on the wall (class metadata + a short-lived stack frame).

| | `new Foo()` | `Foo::bar()` |
|---|-------------|--------------|
| Class metadata | Loaded once (shared) | Loaded once (shared) |
| Object header + property slots | **Allocated per instance** (heap) | **None** |
| `$this` / instance state | Yes | No |
| Lifetime | Until no references remain | Call frame lives for the call only |
| Extra RAM if you do it 10,000 times | 10,000 objects | Still one class; 10,000 short-lived stack frames |

**Instantiating:**

- The class is loaded **once** (autoload + `zend_class_entry`).
- Each `new` allocates a **new object** on the heap: object header, property storage, GC refcount.
- That RAM stays until the object is unreachable and GC / refcount frees it.

**Static call:**

- Same shared class metadata (already in RAM after first load).
- PHP allocates a **call stack frame** for arguments and locals, then pops it when the method returns.
- No per-call object. Static **properties** (if any) live once on the class, not per call.

Important nuance: if the static method itself does `new self()` inside, you still pay for that object. The *call* is cheap; factories still allocate.

I do not choose static methods to “save RAM” in a Laravel app. I choose them when there is no instance state. Object overhead is tiny compared to DB queries.

**Official** ([PHP Manual: The Basics](https://www.php.net/manual/en/language.oop5.basic.php) + [Static Keyword](https://www.php.net/manual/en/language.oop5.static.php)):

```php
$a = new SimpleClass();
$b = new SimpleClass(); // second object — its own $var on the heap

Foo::aStaticMethod();   // no object; class table + call frame only
```

**In production:**

```php
$orders = [];
foreach (range(1, 10_000) as $i) {
    $orders[] = new Order($i); // 10k heap objects (e.g. hydrating a report badly)
}

Order::pendingStatus(); // one class in RAM; do not do this 10k times as “optimization”
```

**Watch out:**

Do not choose static methods to “save RAM” in Laravel. Object overhead is tiny next to a DB query. If the static method `new`s inside, you still pay for the object.

**If they follow up:**

- 10,000 `new Order`? 10,000 heap objects. 10,000 `Order::status()`? Still one class + short frames.
- Why not micro-optimize this? Measure queries first.

> [!TIP]
> **One-liner:** `new` allocates a heap object every time; a static call only uses the already-loaded class plus a stack frame. Static is not a performance strategy — it is “no instance state”.

**Source:** [PHP: Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) (no instance) plus [Garbage Collection](https://www.php.net/manual/en/features.gc.php) (objects live on the heap until unreferenced).

**Learn more:**
- [Nikita Popov: Internal value representation in PHP 7 (part 2)](https://www.npopov.com/2015/06/19/Internal-value-representation-in-PHP-7-part-2.html) — object handles and heap storage (PHP 7+ model; still the basis of 8.x)
- [PHP Internals Book: object handlers](https://www.phpinternalsbook.com/php7/classes_objects/object_handlers.html) — what an allocated object actually is (community internals book, not the PHP Manual)
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — each `new` is a separate instance with its own property values

---

[← Previous](./12-static-method-internals.md) · [Topic](./README.md) · [Next →](./14-abstraction.md)
