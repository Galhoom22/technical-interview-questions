# How do you integrate with an external API?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

I treat the external API as a **client behind an interface**, not as random `Http::get` calls in controllers.

**Steps:**

1. **Read their contract** — base URL, auth (Bearer, API key, OAuth), rate limits, error shape.
2. **Config, not code** — `config/services.php` + `.env` (`STRIPE_KEY`, `STRIPE_BASE_URL`). Never commit secrets.
3. **One client class** — e.g. `StripeClient` using Laravel’s HTTP client (Guzzle).
4. **Timeouts, retries, idempotency** — fail fast, retry only idempotent GETs or with an idempotency key.
5. **Map to our DTOs** — do not leak their JSON through our API.
6. **Errors** — translate 4xx/5xx into domain exceptions; log correlation ids; do not dump their body to the user.
7. **Tests** — `Http::fake()` so tests never hit the network.

```php
public function charge(int $cents): string
{
    $response = Http::baseUrl(config('services.stripe.url'))
        ->withToken(config('services.stripe.key'))
        ->timeout(10)
        ->retry(2, 100)
        ->post('/charges', ['amount' => $cents]);

    $response->throw();

    return $response->json('id');
}
```

Async work (slow third parties) goes on a **queue**. Webhooks inbound get their own signed endpoint — that is the other direction of the same integration.

> [!TIP]
> **One-liner:** Wrap the vendor in a client class, put credentials in `.env`, use Laravel `Http` with timeout/retry, map JSON to our types, and fake it in tests.

**Source:** [Laravel: HTTP Client](https://laravel.com/docs/13.x/http-client) — `Http::`, timeout, retry, `throw()`, `Http::fake()`.

**Learn more:**
- [OWASP API Security — Unsafe Consumption of APIs](https://owasp.org/www-project-api-security/) — do not blindly trust third-party JSON (API10)
- [Laravel: Queues](https://laravel.com/docs/13.x/queues) — move slow vendor calls off the request
- [Laravel: Configuration](https://laravel.com/docs/13.x/configuration) — `config/services.php` + `.env`, never commit secrets

---

[← Previous](./08-multipart-update-with-files.md) · [Topic](./README.md)
