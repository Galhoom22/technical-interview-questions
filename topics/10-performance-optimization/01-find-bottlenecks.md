# If a project is suffering from slow performance, what steps would you take to identify and resolve the bottlenecks?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)

**Answer:**

I **measure first**. I do not add Redis, queues, and eager loading at random. The slow part is often one query or one external HTTP call.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Bottleneck | The slow part you **measured**, not a guess. |
| N+1 | One query plus one query per row because a relation was lazy-loaded. |
| Eager loading | `with()` so related rows load in fewer queries (avoid N+1). |
| `EXPLAIN` | The database plan for a query (seq scan vs index). |
| Telescope | Laravel’s first-party request inspector (queries, outgoing HTTP, jobs). |

### 🧠 Analogy

I treat slowness like a **doctor**: measure (Telescope, slow log, EXPLAIN), name the organ (N+1, missing index, Stripe timeout), then operate. Redis and “more RAM” are not the first prescription.

**1. Define “slow”**

- Which URL, which job, which environment?
- Always, or p95 / under load?
- Browser TTFB vs query time vs queue lag — they are different bugs.

**2. Locate the layer**

| Layer | How I look |
|-------|------------|
| SQL | Slow query log, `EXPLAIN`, [Telescope](https://laravel.com/docs/13.x/telescope) (first-party), `DB::listen` |
| App (N+1, CPU) | Telescope; optional profilers: Xdebug / Blackfire / Tideways (third-party) |
| HTTP to others | Timeouts in logs, Telescope outgoing requests |
| Frontend | Not my first guess for a Laravel API, but I check payload size |
| Infra | CPU, RAM, disk I/O, OPcache, PHP-FPM workers |

**3. Fix the usual Laravel bottlenecks (in this order)**

1. **N+1** — `User::with('orders.items')->get()`. Telescope’s query count is the smoking gun. Locally I also enable `Model::preventLazyLoading(! app()->isProduction())` so missed `with()` fails the request instead of hiding.
2. **Missing indexes** — `EXPLAIN` should not `ALL` on a 2M-row table for `where user_id = ?`.
3. **Select * / huge Eloquent graphs** — `select()`, pagination, `chunkById` for jobs.
4. **Cache** what is expensive and stable — config, routes, views (`artisan optimize`), query/result cache, Redis for sessions/hot keys.
5. **Move off the request** — emails, images, reports → queues ([Horizon](https://laravel.com/docs/13.x/horizon) if you use Redis queues).
6. **PHP/runtime** — OPcache on, too many service providers doing work on every request, debug mode off in production.

---

### 📘 Official ([Laravel: Eager Loading](https://laravel.com/docs/13.x/eloquent-relationships#eager-loading))

```php
$books = Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

Without `with('author')` this is N+1: one query for books, then one per author.

### 💼 In production

```php
// before: N+1 on the order-history page
foreach (User::all() as $user) {
    echo $user->orders->count();
}

User::withCount('orders')->paginate(50);

Model::preventLazyLoading(! app()->isProduction());
```

**4. Prove it**

Same endpoint, before/after: query count, time, EXPLAIN. If it is not faster, the guess was wrong — I revert and measure again.

**What I do not do first:** rewrite the app, switch databases, or “add more RAM” without a profile. Hardware is last, after a real bottleneck is named.

---

> [!WARNING]
> Do not add cache, queues, and eager loading at random. If after/before is not faster, revert — the guess was wrong.

> [!NOTE]
> - Usual Laravel #1? N+1 — `with()` / `withCount`, `preventLazyLoading` locally.
> - Then? Indexes (`EXPLAIN`), then cache/queues.

---

> [!TIP]
> **One-liner:** Measure (Telescope/slow log/EXPLAIN), fix N+1 and indexes first, then cache and queues — never optimize from a guess.

**Source:** [Laravel: Eager Loading](https://laravel.com/docs/13.x/eloquent-relationships#eager-loading) — N+1 is the usual Laravel bottleneck; [Preventing Lazy Loading](https://laravel.com/docs/13.x/eloquent-relationships#preventing-lazy-loading) is the Laravel 13 way to catch it; [Telescope](https://laravel.com/docs/13.x/telescope) to *see* query count.

**Learn more:**
- [MySQL: EXPLAIN](https://dev.mysql.com/doc/refman/8.4/en/explain.html) — read the plan before adding indexes
- [PostgreSQL: Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html) — same idea on Postgres
- [Laravel: Deployment / optimization](https://laravel.com/docs/13.x/deployment#optimization) — config/route/view cache, `APP_DEBUG=false`
- [Laravel: Cache](https://laravel.com/docs/13.x/cache) and [Queues](https://laravel.com/docs/13.x/queues) — after the query is already cheap
- [Laravel Horizon](https://laravel.com/docs/13.x/horizon) — Redis queue dashboard (official)

---

[Topic](./README.md)
