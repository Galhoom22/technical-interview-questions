# How do you approach API design, and what tools do you use?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

I design the **contract first**, then implement it.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Contract | The URLs, methods, JSON shapes, and status codes clients can depend on. |
| OpenAPI | Machine-readable HTTP contract (the spec you document against). |
| Form Request | Laravel class that validates (and often authorizes) the incoming request. |
| API Resource | Laravel class that shapes a model into JSON. |
| Versioning | `/api/v1/...` when a public API will break clients later. |

### 🧠 Analogy

API design is writing the **menu before the kitchen**. OpenAPI is the printed menu; Form Requests and API Resources are how Laravel cooks it.

**Approach:**

1. **List use cases** — who calls this, what they must do (mobile app, admin, webhook).
2. **Identify resources** — `users`, `orders`, `order-items`, not a bag of RPC URLs.
3. **Map verbs** — GET list/show, POST create, PATCH update, DELETE.
4. **Agree errors** — 401 / 403 / 404 / 422 with a consistent JSON shape.
5. **Version if public** — `/api/v1/...` when breaking changes are likely.
6. **Auth + limits** — Sanctum/token, throttle, pagination from day one.
7. **Document the contract** — then code against it.

**Tools:**

| Job | Tool |
|-----|------|
| Sketch & share the contract | OpenAPI (Swagger) / Stoplight / Apidog |
| Manual + collection tests | Postman (or Insomnia) |
| Laravel implementation | Form Requests, API Resources, Sanctum |
| Local HTTP | `Http::fake()` in tests, Telescope |
| CI | Postman Newman or PHPUnit/Pest hitting the API |

I do not start from random controller methods. I start from resources, status codes, and the JSON the client will depend on.

---

### 📘 Official ([OpenAPI Specification](https://spec.openapis.org/oas/latest.html))

```yaml
paths:
  /pets:
    get:
      summary: List all pets
      responses:
        '200':
          description: A paged array of pets
```

### 💼 In production

```php
class StoreOrderRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'product_id' => ['required', 'exists:products,id'],
            'qty' => ['required', 'integer', 'min:1'],
        ];
    }
}

class OrderResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'status' => $this->status,
            'total_cents' => $this->total_cents,
        ];
    }
}
```

---

### ⚠️ Watch out

> [!WARNING]
> Do not start from random controller methods. The JSON the client depends on *is* the product. Version public APIs before you break them.

### 💬 If they follow up

> [!NOTE]
> - Tools? OpenAPI + Postman + Pest. Auth? Sanctum.
> - Pagination/throttle? From day one, not “later.”

---

> [!TIP]
> **One-liner:** I design resources and error codes first, document them (OpenAPI/Postman), then implement with Laravel Form Requests, API Resources, and token auth.

**Source:** [OpenAPI Specification](https://spec.openapis.org/oas/latest.html) — the current OAS (the contract-first format this approach documents against).

**Learn more:**
- [Postman: write test scripts](https://learning.postman.com/docs/tests-and-scripts/write-scripts/test-scripts) — collections as a living contract
- [Laravel: Form Request Validation](https://laravel.com/docs/13.x/validation#form-request-validation) — `$request->validated()` as the server-side contract
- [Laravel: Eloquent API Resources](https://laravel.com/docs/13.x/eloquent-resources) — consistent JSON responses
- [Laravel Sanctum](https://laravel.com/docs/13.x/sanctum) — token / SPA auth for that API

---

[← Previous](./07-restful-api.md) · [Topic](./README.md) · [Next →](./09-external-api-integration.md)