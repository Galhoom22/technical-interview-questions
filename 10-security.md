# 🔒 Security

## Q1. How do you secure endpoints defined in `web.php` and `api.php`?

**Answer:**

Same goal — **authenticate, authorize, validate, throttle** — different tools because `web` is a browser session and `api` is usually a token.

| | `web.php` | `api.php` |
|---|-----------|-----------|
| Identity | Session cookie (`auth`) | Token (`auth:sanctum` / Passport) |
| CSRF | Required on POST/PUT/PATCH/DELETE | Not used for stateless Bearer tokens |
| XSS / cookies | `HttpOnly`, `Secure`, `SameSite` | N/A for pure Bearer |
| CORS | Same origin by default | Must be explicit for browser SPAs |
| Typical extra | `verified`, `password.confirm` | `throttle:api`, abilities/scopes |

**Web routes**

```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('/orders', [OrderController::class, 'index']);
    Route::post('/orders', [OrderController::class, 'store']); // CSRF via web group
});
```

The `web` group already applies session, cookie encryption, and CSRF. I still add `auth`, policies/`can`, and Form Request validation. I never disable CSRF “to make the AJAX easier” without a real alternative (Sanctum SPA with CSRF cookie is the alternative).

**API routes**

```php
Route::middleware(['auth:sanctum', 'throttle:api'])->group(function () {
    Route::apiResource('orders', OrderController::class);
});
```

- Public endpoints stay **outside** `auth:sanctum` (login, health) and are throttled harder.
- Authorization is **policies / gates**, not “the token exists so they can do anything”.
- Mass assignment: `$fillable` / `$guarded`, never `$request->all()` into `create()`.
- HTTPS only; tokens in `Authorization: Bearer`, not in query strings.
- Hide internals: don’t leak stack traces; `APP_DEBUG=false`.

**Both**

- Validate every input (Form Requests).
- `authorize()` in the Form Request or `$this->authorize()` in the controller.
- Rate limit login and sensitive POSTs (`429`).
- Signed URLs for one-click email actions.

> [!TIP]
> **One-liner:** `web.php` = session + CSRF + `auth`. `api.php` = Sanctum/token + throttle + CORS. Both still need policies and validation.

---

## Q2. If a guest user adds products to their cart, how do you ensure those items are retained in their cart after they log in?

**Answer:**

The guest cart lives on the **session** (or a signed cookie / `cart_token`). The logged-in cart lives in the **database** on `user_id`. On login I **merge** session → user, then drop the guest cart.

**While guest**

- Cart key: `session()->getId()` (or a UUID in a cookie).
- Rows: `cart_items` with `session_id` (and `user_id` nullable).

**On login** (Laravel `Login` event or in the login action, after auth succeeds):

1. Load guest items by `session_id`.
2. Load (or create) the user’s cart.
3. **Merge rules** (agree with the business, then code them):
   - Same variant already in user cart → **add quantities** (cap at stock).
   - New variant → **move** the row to `user_id`, clear `session_id`.
   - Invalid / out of stock → skip or clamp, do not crash login.
4. Delete leftover guest rows. `session()->regenerate()` (Laravel already does this on login) so the old session id cannot be reused.

```php
public function handle(Login $event): void
{
    $guestId = session()->get('guest_cart_id'); // stored before regenerate
    $this->carts->mergeGuestCart($event->user, $guestId);
}
```

**Gotchas I mention:**

- Session regenerates on login — **capture the guest cart id before** regenerate, or merge in a listener that still has the old data.
- Two devices: merge is per login of *this* session, not a global “steal the other phone’s cart”.
- Prices: cart may store `variant_id` + qty only; re-price from catalog at checkout.
- Guests who never log in: expire old `session_id` carts with a scheduled command.

I do not keep the cart only in `$_SESSION` arrays if it must survive login across servers — persist `cart_items` and attach `user_id` on merge.

> [!TIP]
> **One-liner:** Persist the guest cart by session id, then on `Login` merge quantities into the user’s cart and delete the guest rows — watch session regeneration so you don’t lose the id.
