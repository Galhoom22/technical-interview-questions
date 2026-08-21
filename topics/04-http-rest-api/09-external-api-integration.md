# How do you integrate with an external API?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

I treat the external API as a **client behind an interface**, not as random `Http::get` calls in controllers.

**Key terms:**

| Term | Plain meaning |
|------|----------------|
| HTTP client | Laravel `Http` facade (Guzzle) for calling other APIs. |
| Timeout | Fail the outbound call if the vendor is slow. |
| Idempotency key | A header so a retried POST does not charge twice. |
| `Http::fake()` | Tests never hit the network. |
| DTO | Our type. Map vendor JSON into it — do not leak their shape. |

**Analogy:**

Stripe is a **vendor behind a locked door**. Checkout talks to *our* `Payable`. Credentials live in `.env`. Tests knock on a fake door (`Http::fake()`), not the real one.

**Steps:**

1. **Read their contract** — base URL, auth (Bearer, API key, OAuth), rate limits, error shape.
2. **Config, not code** — `config/services.php` + `.env` (`STRIPE_KEY`, `STRIPE_BASE_URL`). Never commit secrets.
3. **One client class** — e.g. `StripeClient` using Laravel’s HTTP client (Guzzle).
4. **Timeouts, retries, idempotency** — fail fast, retry only idempotent GETs or with an idempotency key.
5. **Map to our DTOs** — do not leak their JSON through our API.
6. **Errors** — translate 4xx/5xx into domain exceptions; log correlation ids; do not dump their body to the user.
7. **Tests** — `Http::fake()` so tests never hit the network.

**Official** ([Laravel: HTTP Client](https://laravel.com/docs/13.x/http-client)):

```php
$response = Http::withToken($token)
    ->timeout(3)
    ->get('http://example.com/users');

$response->throw();
$users = $response->json();
```

```php
Http::fake([
    'example.com/*' => Http::response(['id' => 1], 200),
]);
```

**In production:**

```php
public function charge(int $cents): string
{
    $response = Http::baseUrl(config('services.stripe.url'))
        ->withToken(config('services.stripe.key'))
        ->timeout(10)
        ->retry(2, 100)
        ->post('/charges', ['amount' => $cents, 'currency' => 'usd']);

    $response->throw();

    return $response->json('id'); // map to our type — do not return Stripe JSON as-is
}
```

Async work (slow third parties) goes on a **queue**. Webhooks inbound get their own signed endpoint — that is the other direction of the same integration.

**Watch out:**

Never commit keys. Never `Http::get` from a controller in five places. Never return Stripe’s JSON as your API. Always set a timeout.

**If they follow up:**

- Retries? Idempotent GET, or POST with an idempotency key.
- Slow vendor? Queue. Inbound? Signed webhook.

> [!TIP]
> **One-liner:** Wrap the vendor in a client class, put credentials in `.env`, use Laravel `Http` with timeout/retry, map JSON to our types, and fake it in tests.

**Source:** [Laravel: HTTP Client](https://laravel.com/docs/13.x/http-client) — `Http::`, timeout, retry, `throw()`, `Http::fake()`.

**Learn more:**
- [OWASP API Security — Unsafe Consumption of APIs](https://owasp.org/www-project-api-security/) — do not blindly trust third-party JSON (API10)
- [Laravel: Queues](https://laravel.com/docs/13.x/queues) — move slow vendor calls off the request
- [Laravel: Configuration](https://laravel.com/docs/13.x/configuration) — `config/services.php` + `.env`, never commit secrets

---

[← Previous](./08-multipart-update-with-files.md) · [Topic](./README.md)
