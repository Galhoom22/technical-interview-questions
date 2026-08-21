# What is the `api.php` file?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

`routes/api.php` holds **stateless API** routes. They are usually prefixed with `/api` and loaded in the `api` middleware group (throttling, JSON, no session CSRF).

**Key terms:**

| Term | Plain meaning |
|------|----------------|
| `api.php` | Stateless API routes. Usually `/api`, `api` middleware group, JSON. |
| Sanctum | Laravel 13 default API auth (Bearer tokens, or SPA cookies). |
| Stateless | The server does not keep a login session. Each request carries the token. |
| Throttle | Rate limit — too many requests → `429`. |
| `install:api` | Artisan command that creates `api.php`, installs Sanctum, and registers the file. |

**Official** ([Laravel Sanctum](https://laravel.com/docs/13.x/sanctum) — `php artisan install:api`):

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

```bash
php artisan install:api
```

**In production:**

```php
Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::apiResource('orders', OrderController::class);
    Route::post('orders/{order}/refund', [RefundController::class, 'store']);
});
```

| | `web.php` | `api.php` |
|---|-----------|-----------|
| Client | Browser | App / SPA / third party |
| Auth | Session cookie | Token (**Sanctum** is Laravel 13’s default via `install:api`; Passport is optional OAuth2) |
| CSRF | Yes (session / SPA cookie) | No for **stateless Bearer** tokens. First-party Sanctum **SPA** still uses CSRF cookies. |
| URL | `/dashboard` | `/api/user` |
| Typical response | HTML | JSON |

**Laravel 13:** `api.php` is not on a fresh install. You add it with `php artisan install:api`, which also installs Sanctum and wires the `api` middleware group.

> [!TIP]
> **One-liner:** `api.php` is for JSON APIs — `/api` prefix, rate limiting, Sanctum token (or SPA cookies). Stateless Bearer skips CSRF; Sanctum SPA still uses CSRF cookies.

**Source:** [Laravel: Routing](https://laravel.com/docs/13.x/routing) — API routes / `api` middleware group; token auth in [Laravel Sanctum](https://laravel.com/docs/13.x/sanctum).

**Learn more:**
- [Laravel: Directory Structure](https://laravel.com/docs/13.x/structure) — `routes/api.php` and `install:api`
- [Laravel: Rate Limiting](https://laravel.com/docs/13.x/rate-limiting) — `throttle:api` / `RateLimiter` in Laravel 13
- [Laravel Passport](https://laravel.com/docs/13.x/passport) — optional OAuth2 when you need authorization-server flows (not the Laravel 13 default)
- [OWASP API Security](https://owasp.org/www-project-api-security/) — what “secure the API surface” means beyond Laravel defaults

---

[← Previous](./03-web-php.md) · [Topic](./README.md) · [Next →](./05-eloquent-relationships.md)
