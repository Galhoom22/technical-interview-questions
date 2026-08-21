# What is the difference between Eloquent ORM and the Query Builder?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

**Query Builder** (`DB::table(...)`) builds SQL and returns plain data (arrays / `stdClass`). **Eloquent** is an ORM on top of that builder: each row is a **model** with relationships, casts, events, and mutators.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Query Builder | SQL in PHP (`DB::table`). Returns arrays / `stdClass`, not models. |
| Eloquent | ORM on top of the Query Builder. Each row is a model (relations, casts, events). |
| Hydration | Turning SQL rows into model objects. |
| ORM | Object-Relational Mapper — tables as classes, rows as objects. |

### 🧠 Analogy

Query Builder is **SQL in PHP** (cheap rows). Eloquent is that builder **plus a model** — relations, casts, events. Eloquent sits on top; it is not a second database.

| Aspect | Query Builder | Eloquent |
|--------|---------------|----------|
| Entry | `DB::table('users')` | `User::query()` |
| Result | Arrays / objects, not models | `User` models / collections |
| Relationships | You join by hand | `with()`, relation methods |
| Casts / hidden / appends | No | Yes |
| Model events / observers | No | Yes |
| Overhead | Lower | Higher (hydration) |
| Best for | Reports, aggregates, bulk SQL | Domain entities, CRUD |

---

### 📘 Official ([Laravel: Query Builder](https://laravel.com/docs/13.x/queries) + [Eloquent](https://laravel.com/docs/13.x/eloquent))

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->get();

$flights = Flight::all();
```

### 💼 In production

```php
$emails = DB::table('users')
    ->where('newsletter', true)
    ->pluck('email'); // marketing export — no User models

$orders = Order::with('items')
    ->where('user_id', $request->user()->id)
    ->latest()
    ->paginate(20); // account page — need models, casts, policies
```

Eloquent **uses** the Query Builder internally (`User::query()` is a builder). Choosing Eloquent vs `DB::` is about whether you need a model or just SQL.

I use Eloquent for domain work. I drop to Query Builder (or `selectRaw`) for heavy reports, bulk updates, and queries where hydrating thousands of models would be wasteful.

---

### ⚠️ Watch out

> [!WARNING]
> Hydrating 50k Eloquent models for a CSV is the trap. `User::query()` is still the Query Builder underneath.

### 💬 If they follow up

> [!NOTE]
> - When `DB::table`? Reports, aggregates, bulk SQL.
> - Relationships? Stay on Eloquent (`with()`).

---

> [!TIP]
> **One-liner:** Query Builder is SQL in PHP. Eloquent is that builder plus models — relationships, casts, and events. Eloquent sits on top of the Query Builder.

**Source:** [Laravel: Query Builder](https://laravel.com/docs/13.x/queries) and [Eloquent](https://laravel.com/docs/13.x/eloquent) — official split this comparison is based on.

**Learn more:**
- [Eloquent: Relationships](https://laravel.com/docs/13.x/eloquent-relationships) — the main reason to stay on Eloquent
- [Eloquent: Mutators & Casting](https://laravel.com/docs/13.x/eloquent-mutators) — what Query Builder will not do for you
- [Database: Getting Started](https://laravel.com/docs/13.x/database) — connections, `DB::transaction`, logging

---

[← Previous](./06-eloquent-relationships.md) · [Topic](./README.md) · [Next →](./08-factories.md)