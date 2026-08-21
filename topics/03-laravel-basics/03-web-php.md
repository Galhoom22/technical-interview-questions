# What is the `web.php` file?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

`routes/web.php` is where **browser / session** routes live. In a default Laravel app they are loaded in the `web` middleware group.

That group typically gives you:

| Feature | Why it is on web routes |
|---------|-------------------------|
| Cookies + session | Logged-in users, flash data |
| CSRF protection | HTML forms must send a token |
| Cookie encryption | Session cookie is not plaintext |
| `SubstituteBindings` | Route-model binding |

**Official** ([Laravel: Routing](https://laravel.com/docs/13.x/routing) — default `web.php`):

```php
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});
```

Those routes sit in the `web` middleware group (session, cookies, CSRF).

**In production:**

```php
Route::middleware('auth')->group(function () {
    Route::get('/account/orders', [OrderHistoryController::class, 'index'])
        ->name('account.orders');

    Route::post('/account/orders/{order}/cancel', [CancelOrderController::class, 'store'])
        ->name('account.orders.cancel'); // browser form + CSRF token
});
```

These routes are for Blade / Inertia / Livewire pages, not for a stateless mobile API.

> [!TIP]
> **One-liner:** `web.php` holds session-based browser routes — cookies, CSRF, and auth via the `web` middleware group.

**Source:** [Laravel: Routing](https://laravel.com/docs/13.x/routing) — web routes and the `web` middleware group; CSRF detail in [Laravel: CSRF Protection](https://laravel.com/docs/13.x/csrf).

**Learn more:**
- [Laravel: Directory Structure](https://laravel.com/docs/13.x/structure) — where `routes/web.php` lives
- [Laravel: Session](https://laravel.com/docs/13.x/session) — the session store those routes use
- [Laravel: Authentication](https://laravel.com/docs/13.x/authentication) — `auth` middleware on web routes

---

[← Previous](./02-routing.md) · [Topic](./README.md) · [Next →](./04-api-php.md)
