# What is Middleware, and what are its primary use cases?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Middleware is a **filter around the HTTP request**. It can inspect, reject, or enrich the request before the controller, and it can tweak the response on the way out.

**🔑 Key terms:**

| Term | Plain meaning |
|------|----------------|
| Middleware | A filter around the HTTP request (and the response on the way out). |
| `handle` | The middleware method. Call `$next($request)` to continue, or abort/redirect. |
| Global middleware | Runs on every request. |
| Route middleware | Runs only on the routes you attach it to (or a group). |

**🧠 Analogy:**

Middleware is the **security line at the door**: check ID, stamp the ticket, then the controller. On the way out it can still add a header.

```
request → middleware → middleware → controller → middleware → response
```

**📘 Official** ([Laravel: Middleware](https://laravel.com/docs/13.x/middleware)):

```php
class EnsureTokenIsValid
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('/home');
        }

        return $next($request);
    }
}
```

**💼 In production:**

```php
class EnsureCheckoutNotLocked
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user()?->checkout_locked_at) {
            abort(423, 'Checkout is locked while payment is processing.');
        }

        return $next($request);
    }
}

Route::post('/checkout', CheckoutController::class)
    ->middleware(['auth', 'checkout.unlocked']);
```

**Primary use cases:**

| Use case | Example |
|----------|---------|
| Authentication | `auth`, `auth:sanctum` |
| Authorization | `can:update,post` |
| CSRF | `ValidateCsrfToken` on web |
| Rate limiting | `throttle:api` |
| Localization | set locale from header |
| Logging / CORS | log request, add CORS headers |
| Maintenance | `PreventRequestsDuringMaintenance` |

Registration:

- **Global** — every request
- **Group** — `web` vs `api`
- **Route** — one route or a `Route::middleware([...])` group

Laravel **13** registers middleware in `bootstrap/app.php` (`withMiddleware()`), not `Http/Kernel.php`.

**⚠️ Watch out:**

Forgetting `$next($request)` drops the request. Laravel 13: `bootstrap/app.php`, not `Http/Kernel.php`. Do not stuff validation into middleware — Form Request is the layer.

**💬 If they follow up:**

- Global vs route? Every request vs the routes you attach.
- CSRF? `web` group. `auth:sanctum`? API group.

> [!TIP]
> **One-liner:** Middleware is request/response middleware — auth, CSRF, throttle, CORS — that runs before (and after) the controller.

**Source:** [Laravel: Middleware](https://laravel.com/docs/13.x/middleware) — `handle`, global vs group vs route middleware.

**Learn more:**
- [Laravel: CSRF Protection](https://laravel.com/docs/13.x/csrf) — the most important `web` middleware
- [Laravel: Authentication](https://laravel.com/docs/13.x/authentication#protecting-routes) — `auth` / `auth:sanctum`
- [Laravel: Form Request Validation](https://laravel.com/docs/13.x/validation#form-request-validation) — validation as its own layer, not inside the controller
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) — why API middleware often adds CORS headers

---

[← Previous](./05-eloquent-relationships.md) · [Topic](./README.md) · [Next →](./07-eloquent-vs-query-builder.md)
