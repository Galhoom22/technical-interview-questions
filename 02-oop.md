# 🧩 OOP (Object-Oriented Programming)

## Q1. What is OOP?

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
- [PHP: The Right Way — Getting Started](https://phptherightway.com/) — modern PHP practices around OOP, autoloading, and packages
- [Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — `public` / `protected` / `private` (encapsulation in the language)

---

## Q2. What are the four core principles of OOP?

**Answer:**

| Principle | Meaning | In practice |
|-----------|---------|-------------|
| **Encapsulation** | Hide internal state; expose a small public API | `private` properties, public methods |
| **Abstraction** | Show what something does, hide how | Interfaces, abstract classes |
| **Inheritance** | A child reuses / specializes a parent | `class Admin extends User` |
| **Polymorphism** | Same message, different behavior | `Payable[]` of `Invoice` and `Salary` |

```php
interface Payable
{
    public function pay(): void;
}

class Invoice implements Payable
{
    public function pay(): void { /* charge customer */ }
}

class Salary implements Payable
{
    public function pay(): void { /* pay employee */ }
}

function settle(Payable $item): void
{
    $item->pay(); // same call, different implementation
}
```

I mention inheritance last on purpose. In PHP I prefer **composition + interfaces** over deep class trees. Inheritance is a tool, not the goal.

> [!TIP]
> **One-liner:** Encapsulation hides state, abstraction hides complexity, inheritance shares structure, polymorphism lets different classes answer the same message.

**Source:** [PHP: Classes and Objects](https://www.php.net/manual/en/language.oop5.php) — the four principles map onto these language features (visibility, abstract/interface, `extends`, interfaces).

**Learn more:**
- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — `extends`, overriding, `parent::`
- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — polymorphism via a shared contract
- [Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) — abstract classes as a shared family

---

## Q3. What is a Class?

**Answer:**

A class is the **blueprint**. An object (instance) is a **value built from that blueprint**.

The class defines:

- properties (state)
- methods (behavior)
- visibility (`public` / `protected` / `private`)
- how instances are constructed

```php
class User
{
    public function __construct(
        public string $name,
        public string $email,
    ) {}

    public function mention(): string
    {
        return '@' . $this->name;
    }
}

$user = new User('Omar', 'omar@example.com'); // instance
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

## Q4. What is an Interface, and why is it used?

**Answer:**

An interface is a **contract**: a list of public methods a class **must** implement. It has no state (historically no properties) and does not care *how* the work is done.

```php
interface Mailer
{
    public function send(string $to, string $subject, string $body): void;
}

class SmtpMailer implements Mailer
{
    public function send(string $to, string $subject, string $body): void
    {
        // SMTP implementation
    }
}

class LogMailer implements Mailer
{
    public function send(string $to, string $subject, string $body): void
    {
        logger()->info("Mail to {$to}: {$subject}");
    }
}
```

**Why we use it:**

- **Decoupling** — controllers depend on `Mailer`, not `SmtpMailer`.
- **Polymorphism** — any implementation can be swapped (real SMTP in prod, fake in tests).
- **Multiple contracts** — a class can `implement` many interfaces; it can `extend` only one class.
- **Clear API** — the interface is the public surface other code is allowed to rely on.

In Laravel this is the Service Container pattern: bind `Mailer::class` to `SmtpMailer::class`.

> [!TIP]
> **One-liner:** An interface is a contract of public methods. You use it so code depends on *behavior*, not a specific class — which makes swapping and testing easy.

**Source:** [PHP: Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — official contract, `implements`, and multiple interfaces.

**Learn more:**
- [Laravel: Service Container](https://laravel.com/docs/container) — binding an interface to a class (the pattern used in real Laravel apps)
- [Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) — when a partial implementation is a better fit than a pure interface
- [PHP 8.4 Release](https://www.php.net/releases/8.4/en.php) — interface property hooks (newer language change interviewers may mention)

---

## Q5. What is a Trait?

**Answer:**

A trait is a reuse tool for **horizontal** sharing of methods (and properties). PHP copies the trait’s members into the class at compile time. It is not inheritance and not an interface.

```php
trait HasSlug
{
    public function slug(string $value): string
    {
        return strtolower(str_replace(' ', '-', $value));
    }
}

class Article
{
    use HasSlug;
}

(new Article())->slug('Hello World'); // "hello-world"
```

If two traits define the same method, you must resolve the conflict with `insteadof` and `as`.

```php
class Example
{
    use TraitA, TraitB {
        TraitA::log insteadof TraitB;
        TraitB::log as logFromB;
    }
}
```

**When I use traits:** small, focused behavior used in several classes that do **not** share a parent (`HasFactory` in Laravel is the classic example).

**When I avoid them:** as a dumping ground for unrelated methods. If the trait needs a lot of hidden state, it probably wants to be a collaborator object instead (composition).

> [!TIP]
> **One-liner:** A trait is copy-paste reuse managed by the language — a way to share methods across classes without multiple inheritance.

**Source:** [PHP: Traits](https://www.php.net/manual/en/language.oop5.traits.php) — official `use`, conflict resolution (`insteadof` / `as`), and the “not inheritance” wording.

**Learn more:**
- [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) — what traits are *not* (a second parent class)
- [Laravel Eloquent: factories](https://laravel.com/docs/eloquent-factories) — `HasFactory` is the trait you already use in Laravel
- [Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — how trait members become part of the class

---

## Q6. How do you implement multiple inheritance in PHP?

**Answer:**

You don’t — **PHP allows a class to extend only one parent**. Multiple inheritance of classes is not supported.

What you do instead:

| Need | Tool |
|------|------|
| Multiple *contracts* | `implements InterfaceA, InterfaceB` |
| Reuse *implementation* across unrelated classes | `use TraitA, TraitB` |
| Reuse *implementation* in a family of types | Single inheritance (`extends`) |
| Reuse by *delegation* | Composition — inject another object |

```php
interface Auditable
{
    public function auditId(): string;
}

interface Cacheable
{
    public function cacheKey(): string;
}

class Report extends BaseReport implements Auditable, Cacheable
{
    use LogsActivity; // shared implementation

    public function auditId(): string { return (string) $this->id; }
    public function cacheKey(): string { return 'report:'.$this->id; }
}
```

If an interviewer says “so traits are multiple inheritance?”, I correct it: traits are compiler-assisted copy-paste, not a second parent class. There is no diamond-problem of *types* — only method-name conflicts you must resolve explicitly.

> [!TIP]
> **One-liner:** PHP has no multiple class inheritance. I combine one `extends`, many `implements`, traits for shared code, and composition when behavior belongs to another object.

**Source:** [PHP: Traits](https://www.php.net/manual/en/language.oop5.traits.php) — PHP’s documented substitute for multiple inheritance of *implementation*; plus [Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php) (single `extends` only).

**Learn more:**
- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — multiple *contracts* via `implements A, B`
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — composition: hold another object as a property instead of extending it
- [Laravel: Service Container](https://laravel.com/docs/container) — injecting collaborators instead of inheriting them

---

## Q7. What is a Destructor in PHP, and what is its purpose?

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
- [PHP Internals Book: object handlers](https://www.phpinternalsbook.com/php7/classes_objects/object_handlers.html) — `dtor_obj` vs `free_obj` (engine view of destruction)
- [Magic Methods](https://www.php.net/manual/en/language.oop5.magic.php) — where `__destruct` sits among the other hooks

---

## Q8. Can a constructor be private?

**Answer:**

Yes. A `private` constructor means **`new ClassName()` is only legal inside that class**. Code outside cannot instantiate it directly. `protected` allows children, but still blocks the outside world.

```php
class Uuid
{
    private function __construct(private string $value) {}

    public static function fromString(string $value): self
    {
        // validate, then:
        return new self($value);
    }
}

Uuid::fromString('…'); // ok
new Uuid('…');         // Error: private constructor
```

**Why you do this:**

- Named constructors / factories (`fromString`, `fromArray`) that validate first
- Singleton (I mention it because they ask; I rarely use it)
- Forcing all creation through a factory or the container

> [!TIP]
> **One-liner:** Yes. Private constructors block `new` from outside and force creation through a static factory (or similar) inside the class.

**Source:** [PHP: Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — methods (including `__construct`) can be `private` / `protected`; plus [Constructors](https://www.php.net/manual/en/language.oop5.decon.php).

**Learn more:**
- [Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — named constructors / factories as `public static function fromString()`
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — `new self()` / `new static()` from inside the class
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `new static()` in a parent factory used by a child

---

## Q9. How do you instantiate an object from a class? Explain the step-by-step process of how the object is built and what handles its creation.

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
- [Nikita Popov: Internal value representation in PHP 7 (part 2)](https://www.npopov.com/2015/06/19/Internal-value-representation-in-PHP-7-part-2.html) — objects, handles, and heap layout
- [PHP Internals Book: object handlers](https://www.phpinternalsbook.com/php7/classes_objects/object_handlers.html) — `get_constructor` and object creation hooks

---

## Q10. If a constructor is private, how can you instantiate an object of that class?

**Answer:**

You instantiate it **from inside the class**, usually with a **static factory** (or `getInstance()` for a singleton). `new self(...)` / `new static(...)` is allowed there because it is the same class.

```php
class Database
{
    private static ?self $instance = null;

    private function __construct(private string $dsn) {}

    public static function getInstance(): self
    {
        return self::$instance ??= new self(config('database.dsn'));
    }
}

Database::getInstance();
```

Other legitimate paths:

- A **public static factory**: `Order::fromCart($cart)` → `return new self(...)`.
- A **friend class in the same file** cannot bypass it — visibility is per class, not per file.
- **Reflection** (`ReflectionClass::newInstanceWithoutConstructor()`) can cheat. That is for frameworks/tests, not production domain code.

A **child class** cannot call a **private** parent constructor. If subclasses should exist, the constructor should be `protected`.

> [!TIP]
> **One-liner:** Call a public static method on the class; that method is allowed to `new self()`. Outside code never uses `new` directly.

**Source:** [PHP: Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — `private` constructor is visible only inside the same class, so a public static factory can still `new self()`.

**Learn more:**
- [Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — factory methods without an instance
- [Constructors and Destructors](https://www.php.net/manual/en/language.oop5.decon.php) — `protected` constructor if subclasses should exist
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `new static()` vs `new self()` in factories

---

## Q11. What is the difference between static and non-static methods?

**Answer:**

| Aspect | Instance (non-static) | Static |
|--------|----------------------|--------|
| Belongs to | An object | The class |
| Call | `$user->name()` | `User::name()` or `static::name()` |
| `$this` | Yes — current instance | No — using `$this` throws |
| State | Instance properties | Only static properties / arguments |
| Typical use | Behavior that needs this object’s data | Factories, utilities, named constructors |

```php
class Counter
{
    private int $value = 0;
    private static int $created = 0;

    public function __construct()
    {
        self::$created++;
    }

    public function increment(): void   // needs $this
    {
        $this->value++;
    }

    public static function created(): int  // no instance needed
    {
        return self::$created;
    }
}

$c = new Counter();
$c->increment();
Counter::created();
```

Static methods are not “faster OOP”. They are functions namespaced on a class. Too many of them usually means the design wanted a service object instead.

> [!TIP]
> **One-liner:** Instance methods run on an object and use `$this`. Static methods run on the class, have no `$this`, and cannot use instance state.

**Source:** [PHP: Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — official rules for static methods, static properties, and “no `$this`”.

**Learn more:**
- [Paamayim Nekudotayim (`::`)](https://www.php.net/manual/en/language.oop5.paamayim-nekudotayim.php) — `ClassName::method()` vs `$obj->method()`
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `self::` vs `static::`
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — `$this` on instance methods

---

## Q12. A static method does not require an object instance to run. How is this managed internally?

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

**Source:** [PHP: Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) — userland rules; internals: [PHP Internals Book: object handlers](https://www.phpinternalsbook.com/php7/classes_objects/object_handlers.html) (`get_method` on the class, not an instance).

**Learn more:**
- [Late Static Bindings](https://www.php.net/manual/en/language.oop5.late-static-bindings.php) — `static::class` is the *called* class, with no object
- [Nikita Popov: Internal value representation in PHP 7 (part 2)](https://www.npopov.com/2015/06/19/Internal-value-representation-in-PHP-7-part-2.html) — zvals vs objects in the engine
- [Autoloading Classes](https://www.php.net/manual/en/language.oop5.autoload.php) — the class entry is loaded once, then reused for every static call

---

## Q13. What is the difference in memory (RAM) allocation between instantiating an object and calling a static method?

**Answer:**

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

> [!TIP]
> **One-liner:** `new` allocates a heap object every time; a static call only uses the already-loaded class plus a stack frame. Static is not a performance strategy — it is “no instance state”.

**Source:** [PHP: Static Keyword](https://www.php.net/manual/en/language.oop5.static.php) (no instance) plus [Garbage Collection](https://www.php.net/manual/en/features.gc.php) (objects live on the heap until unreferenced).

**Learn more:**
- [Nikita Popov: Internal value representation in PHP 7 (part 2)](https://www.npopov.com/2015/06/19/Internal-value-representation-in-PHP-7-part-2.html) — object handles and heap storage
- [PHP Internals Book: object handlers](https://www.phpinternalsbook.com/php7/classes_objects/object_handlers.html) — what an allocated object actually is
- [The Basics](https://www.php.net/manual/en/language.oop5.basic.php) — each `new` is a separate instance with its own property values

---

## Q14. What is the concept of Abstraction?

**Answer:**

Abstraction means exposing **what** something does and hiding **how** it does it. Callers depend on a small, stable interface; the messy details stay inside.

Two layers people mix up:

- **Abstraction (the idea)** — a `PaymentGateway` with `charge()` so the checkout code never talks to Stripe HTTP APIs.
- **Abstract class (the keyword)** — a language tool that *helps* you build that idea, alongside interfaces.

```php
abstract class PaymentGateway
{
    abstract public function charge(int $cents): string;

    public function receipt(string $id): string
    {
        return "Receipt {$id}";
    }
}

class StripeGateway extends PaymentGateway
{
    public function charge(int $cents): string
    {
        // HTTP call to Stripe — hidden from checkout
        return 'ch_123';
    }
}
```

Checkout calls `charge()`. It does not know about API keys, retry policy, or JSON. That is abstraction.

> [!TIP]
> **One-liner:** Abstraction hides implementation behind a simple public API so callers depend on *what* happens, not *how*.

**Source:** [PHP: Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) — official `abstract class` / `abstract` methods (the language tool behind the idea).

**Learn more:**
- [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — abstraction with a contract and no shared code
- [Visibility](https://www.php.net/manual/en/language.oop5.visibility.php) — hide `private` details behind a small public API
- [Laravel: Service Container](https://laravel.com/docs/container) — depending on an abstract type in a real app

---

## Q15. What is the difference between an Abstract Class and an Interface?

**Answer:**

Both define a contract. An **abstract class** can also ship shared code and state. An **interface** is only a contract (multiple allowed).

| Aspect | Abstract class | Interface |
|--------|----------------|-----------|
| Instantiated? | No | No |
| Methods | Abstract + concrete | Signatures (no body, historically) |
| Properties / constructor | Yes | No (PHP 8.4 adds property hooks on interfaces) |
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
- [PHP 8.4 Release](https://www.php.net/releases/8.4/en.php) — property hooks on interfaces (the “no properties” rule is changing)
- [Traits](https://www.php.net/manual/en/language.oop5.traits.php) — a third option when you want reuse without an abstract parent

---

## Q16. Can you instantiate an object directly from an Abstract Class?

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
