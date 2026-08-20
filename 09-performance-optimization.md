# ⚡ Performance & Optimization

## Q1. If a project is suffering from slow performance, what steps would you take to identify and resolve the bottlenecks?

**Answer:**

I **measure first**. I do not add Redis, queues, and eager loading at random. The slow part is often one query or one external HTTP call.

**1. Define “slow”**

- Which URL, which job, which environment?
- Always, or p95 / under load?
- Browser TTFB vs query time vs queue lag — they are different bugs.

**2. Locate the layer**

| Layer | How I look |
|-------|------------|
| SQL | Slow query log, `EXPLAIN`, Laravel Debugbar / Telescope, `DB::listen` |
| App (N+1, CPU) | Telescope, Clockwork, Xdebug profiler / Blackfire / Tideways |
| HTTP to others | Timeouts in logs, Telescope outgoing requests |
| Frontend | Not my first guess for a Laravel API, but I check payload size |
| Infra | CPU, RAM, disk I/O, opcache, PHP-FPM workers |

**3. Fix the usual Laravel bottlenecks (in this order)**

1. **N+1** — `User::with('orders.items')->get()`. Debugbar’s query count is the smoking gun.
2. **Missing indexes** — `EXPLAIN` should not `ALL` on a 2M-row table for `where user_id = ?`.
3. **Select * / huge Eloquent graphs** — `select()`, pagination, `chunkById` for jobs.
4. **Cache** what is expensive and stable — config, routes, views (`artisan optimize`), query/result cache, Redis for sessions/hot keys.
5. **Move off the request** — emails, images, reports → queues (Horizon).
6. **PHP/runtime** — OPcache on, too many service providers doing work on every request, debug mode off in production.

```php
// before: N+1
foreach (User::all() as $user) {
    echo $user->orders->count();
}

// after
User::withCount('orders')->paginate(50);
```

**4. Prove it**

Same endpoint, before/after: query count, time, EXPLAIN. If it is not faster, the guess was wrong — I revert and measure again.

**What I do not do first:** rewrite the app, switch databases, or “add more RAM” without a profile. Hardware is last, after a real bottleneck is named.

> [!TIP]
> **One-liner:** Measure (Telescope/slow log/EXPLAIN), fix N+1 and indexes first, then cache and queues — never optimize from a guess.

**Source:** [Laravel: Eager Loading](https://laravel.com/docs/eloquent-relationships#eager-loading) — N+1 is the usual Laravel bottleneck; plus [Telescope](https://laravel.com/docs/telescope) to *see* query count.

**Learn more:**
- [MySQL: EXPLAIN](https://dev.mysql.com/doc/refman/8.4/en/explain.html) — read the plan before adding indexes
- [PostgreSQL: Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html) — same idea on Postgres
- [Laravel: Deployment / optimization](https://laravel.com/docs/deployment#optimization) — config/route/view cache, `APP_DEBUG=false`
- [Laravel: Cache](https://laravel.com/docs/cache) and [Queues](https://laravel.com/docs/queues) — after the query is already cheap
