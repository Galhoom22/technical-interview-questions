# What is Routing?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Routing maps an **HTTP request** (method + URI) to the code that should handle it: a closure, a controller action, or an invokable controller.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Routing | Map HTTP method + URI to a controller or closure. |
| Route-model binding | `{order}` in the URL becomes an `Order` model. |
| Named route | `->name('orders.show')` so `route()` stays stable if the path changes. |
| Middleware | A filter around the HTTP request (and the response on the way out). |

### 🧠 Analogy

Routing is the **front desk**: “GET `/orders/15` goes to `OrderController@show`.” Named routes are the desk’s nickname so the URL can change without breaking links.

---

### 📘 Official ([Laravel: Routing](https://laravel.com/docs/13.x/routing))

```php
Route::get('/user', [UserController::class, 'index']);
Route::get('/user/{id}', [UserController::class, 'show']);
```

Laravel **13** registers route files in `bootstrap/app.php`:

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
)
```

### 💼 In production

```php
Route::get('/orders/{order}', [OrderController::class, 'show'])
    ->middleware('auth')
    ->name('orders.show');

Route::post('/checkout', [CheckoutController::class, 'store'])
    ->middleware(['auth', 'throttle:6,1']);
```

The router:

1. Matches method + path (and optional name, middleware, where constraints).
2. Runs route middleware.
3. Resolves the controller from the container (dependency injection).
4. Performs route-model binding (`{user}` → `User` model).
5. Returns the response.

Named routes (`->name('users.show')`) keep `route()` URLs from going stale when the path changes.

---

### ⚠️ Watch out

> [!WARNING]
> Do not hardcode `/orders/15` in Blade. Use `route('orders.show', $order)`. Forgetting `web` vs `api` middleware is a common CSRF/session bug.

### 💬 If they follow up

> [!NOTE]
> - Where are files registered in Laravel 13? `bootstrap/app.php` → `withRouting`.
> - `{order}` type-hint? Route-model binding — 404 if missing.

---

> [!TIP]
> **One-liner:** Routing is the map from “HTTP method + URL” to the controller (or closure) that handles that request.

**Source:** [Laravel: Routing](https://laravel.com/docs/13.x/routing) — methods, parameters, named routes, groups, and model binding.

**Learn more:**
- [Laravel: Controllers](https://laravel.com/docs/13.x/controllers) — where the route usually points
- [Laravel: Middleware](https://laravel.com/docs/13.x/middleware) — what runs around the route
- [MDN: HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods) — the verbs `Route::get/post/put/patch/delete` wrap

---

[← Previous](./01-migration.md) · [Topic](./README.md) · [Next →](./03-web-php.md)
