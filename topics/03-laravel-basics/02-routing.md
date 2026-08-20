# What is Routing?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Routing maps an **HTTP request** (method + URI) to the code that should handle it: a closure, a controller action, or an invokable controller.

```php
Route::get('/users/{user}', [UserController::class, 'show']);
Route::post('/users', [UserController::class, 'store']);
```

Laravel **13** loads those files from `bootstrap/app.php`:

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )->create();
```

The router:

1. Matches method + path (and optional name, middleware, where constraints).
2. Runs route middleware.
3. Resolves the controller from the container (dependency injection).
4. Performs route-model binding (`{user}` → `User` model).
5. Returns the response.

Named routes (`->name('users.show')`) keep `route()` URLs from going stale when the path changes.

> [!TIP]
> **One-liner:** Routing is the map from “HTTP method + URL” to the controller (or closure) that handles that request.

**Source:** [Laravel: Routing](https://laravel.com/docs/13.x/routing) — methods, parameters, named routes, groups, and model binding.

**Learn more:**
- [Laravel: Controllers](https://laravel.com/docs/13.x/controllers) — where the route usually points
- [Laravel: Middleware](https://laravel.com/docs/13.x/middleware) — what runs around the route
- [MDN: HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods) — the verbs `Route::get/post/put/patch/delete` wrap

---

[← Previous](./01-migration.md) · [Topic](./README.md) · [Next →](./03-web-php.md)
