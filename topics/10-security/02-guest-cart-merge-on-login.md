# If a guest user adds products to their cart, how do you ensure those items are retained in their cart after they log in?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

The guest cart lives on the **session** (or a signed cookie / `cart_token`). The logged-in cart lives in the **database** on `user_id`. On login I **merge** session → user, then drop the guest cart.

**Key terms:**

| Term | Plain meaning |
|------|----------------|
| Guest cart | Basket keyed by session / cookie while the user is not logged in. |
| Merge | On login, move guest lines onto the user’s cart (add qty if the SKU exists). |
| Session regenerate | Laravel rotates the session id on login. Capture the guest cart key **before** that. |
| `Login` event | Fires after credentials succeed. A listener is the right place to merge. |

**Analogy:**

The guest basket is a **sticky note** on this browser. On login you **pour it into the user’s real cart**, add quantities if the SKU was already there, throw the sticky note away — but copy the note *before* Laravel changes the session lock.

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

**Official** ([Laravel: Events](https://laravel.com/docs/13.x/events) + [Authentication](https://laravel.com/docs/13.x/authentication)):

```php
use Illuminate\Auth\Events\Login;

class MergeGuestCart
{
    public function handle(Login $event): void
    {
        // Login fires after credentials succeed; session id is about to rotate
    }
}
```

**In production:**

```php
public function handle(Login $event): void
{
    $guestId = session()->get('guest_cart_id'); // captured before regenerate
    $this->carts->mergeGuestCart($event->user, $guestId);
}
```

**Gotchas I mention:**

- Session regenerates on login — **capture the guest cart id before** regenerate, or merge in a listener that still has the old data.
- Two devices: merge is per login of *this* session, not a global “steal the other phone’s cart”.
- Prices: cart may store `variant_id` + qty only; re-price from catalog at checkout.
- Guests who never log in: expire old `session_id` carts with a scheduled command.

I do not keep the cart only in `$_SESSION` arrays if it must survive login across servers — persist `cart_items` and attach `user_id` on merge.

**Watch out:**

Session regenerates on login — merge with a cart key stored **in** the session, captured before rotate. Do not merge the *wrong* user’s cart (IDOR).

**If they follow up:**

- Same SKU on both? Add quantities, cap at stock.
- Two devices? This login merges *this* session, not the other phone.

> [!TIP]
> **One-liner:** Persist the guest cart by session id, then on `Login` merge quantities into the user’s cart and delete the guest rows — watch session regeneration so you don’t lose the id.

**Source:** [Laravel: Events](https://laravel.com/docs/13.x/events) (`Login` event) plus [Session](https://laravel.com/docs/13.x/session) — login regenerates the session id; merge must use a cart key stored *in* the session payload, not only the old id.

**Learn more:**
- [Laravel: Authentication](https://laravel.com/docs/13.x/authentication) — when `Login` fires relative to session migrate
- [Laravel: Session](https://laravel.com/docs/13.x/session) — `session()->regenerate()` / login session rotation
- [Eloquent Relationships](https://laravel.com/docs/13.x/eloquent-relationships) — `cart` / `cartItems` as `hasMany`
- [OWASP API Security](https://owasp.org/www-project-api-security/) — broken object-level authorization if you merge the *wrong* user’s cart

---

[← Previous](./01-secure-web-and-api-routes.md) · [Topic](./README.md)
