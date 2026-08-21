# How do you secure endpoints defined in `web.php` and `api.php`?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Same goal — **authenticate, authorize, validate, throttle** — different tools because `web` is a browser session and `api` is usually a token.

| | `web.php` | `api.php` |
|---|-----------|-----------|
| Identity | Session cookie (`auth`) | Token (`auth:sanctum`). Passport is optional OAuth2, not the Laravel 13 default. |
| CSRF | Required on POST/PUT/PATCH/DELETE | Not used for **stateless Bearer** tokens. First-party Sanctum **SPA** still uses CSRF cookies. |
| XSS / cookies | `HttpOnly`, `Secure`, `SameSite` | N/A for pure Bearer |
| CORS | Same origin by default | Must be explicit for browser SPAs |
| Typical extra | `verified`, `password.confirm` | `throttle:api`, abilities/scopes |

**Official** ([Laravel: CSRF](https://laravel.com/docs/13.x/csrf) + [Sanctum](https://laravel.com/docs/13.x/sanctum)):

```html
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

**In production:**

```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::post('/orders', [OrderController::class, 'store']); // session + CSRF
});

Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::apiResource('orders', OrderController::class);
});

$order = Order::create($request->validated()); // never $request->all()
```

The `web` group already applies session, cookie encryption, and CSRF. I still add `auth`, policies/`can`, and Form Request validation.

- Public endpoints stay **outside** `auth:sanctum` (login, health) and are throttled harder.
- Authorization is **policies / gates**, not “the token exists so they can do anything”.
- Mass assignment: Laravel 13 `#[Fillable([...])]` (or `$fillable` / `$guarded`). Persist **`$request->validated()`**, never `$request->all()` into `create()`.
- Locally: `Model::preventSilentlyDiscardingAttributes(! app()->isProduction())` so unfillable fields throw instead of disappearing.
- HTTPS only; tokens in `Authorization: Bearer`, not in query strings.
- Hide internals: don’t leak stack traces; `APP_DEBUG=false`.

**Both**

- Validate every input (Form Requests).
- `authorize()` in the Form Request or `$this->authorize()` in the controller.
- Rate limit login and sensitive POSTs (`429`).
- Signed URLs for one-click email actions.

> [!TIP]
> **One-liner:** `web.php` = session + CSRF + `auth`. `api.php` = Sanctum/token + throttle + CORS. Both still need policies and validation.

**Source:** [Laravel: CSRF Protection](https://laravel.com/docs/13.x/csrf) (web) and [Laravel Sanctum](https://laravel.com/docs/13.x/sanctum) (API) — official split this answer follows.

**Learn more:**
- [Laravel: Authentication](https://laravel.com/docs/13.x/authentication) — `auth` middleware
- [Laravel: Authorization](https://laravel.com/docs/13.x/authorization) — policies / gates (token is not permission)
- [Laravel: Mass Assignment](https://laravel.com/docs/13.x/eloquent#mass-assignment) — `#[Fillable]`, `$guarded`, `preventSilentlyDiscardingAttributes`
- [Laravel: Form Request Validation](https://laravel.com/docs/13.x/validation#form-request-validation) — `$request->validated()`
- [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html) — HTTPS, tokens, error leakage
- [OWASP Mass Assignment Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html) — never `$request->all()` into `create()`
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) — browser SPAs calling `api.php`
- [Laravel Passport](https://laravel.com/docs/13.x/passport) — only when you need a full OAuth2 server

---

[Topic](./README.md) · [Next →](./02-guest-cart-merge-on-login.md)
