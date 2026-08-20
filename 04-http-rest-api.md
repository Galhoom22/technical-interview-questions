# 🌐 HTTP & REST API

## Q1. What are the different HTTP methods?

**Answer:**

HTTP methods (verbs) tell the server **what kind of operation** the client wants on a resource.

| Method | Intent | Request body | Safe | Idempotent |
|--------|--------|--------------|------|------------|
| `GET` | Read | No (should not) | Yes | Yes |
| `HEAD` | Same as GET, headers only | No | Yes | Yes |
| `POST` | Create, or trigger an action | Yes | No | No |
| `PUT` | Replace the whole resource at a URI | Yes | No | Yes |
| `PATCH` | Partial update | Yes | No | Usually not guaranteed |
| `DELETE` | Remove | Optional | No | Yes |
| `OPTIONS` | Ask which methods are allowed (CORS preflight) | No | Yes | Yes |

**Safe** = no server-side state change. **Idempotent** = repeating the same request leaves the same state (`PUT` / `DELETE` / `GET`). `POST` is not idempotent: two creates can mean two rows.

```http
GET    /api/orders
POST   /api/orders
GET    /api/orders/15
PUT    /api/orders/15
PATCH  /api/orders/15
DELETE /api/orders/15
```

> [!TIP]
> **One-liner:** The verbs you use every day are GET (read), POST (create/action), PUT (full replace), PATCH (partial update), DELETE (remove), plus HEAD and OPTIONS.

---

## Q2. What are HTTP status codes, and what are their main categories?

**Answer:**

Status codes are a **3-digit signal** of how the request turned out. The first digit is the category; the body (if any) carries details.

| Range | Category | Meaning |
|-------|----------|---------|
| `1xx` | Informational | Request received, continuing (rare in app APIs) |
| `2xx` | Success | The request worked |
| `3xx` | Redirection | Look somewhere else |
| `4xx` | Client error | The caller did something wrong |
| `5xx` | Server error | We failed |

Codes I actually return:

| Code | When |
|------|------|
| `200 OK` | GET / PATCH success with a body |
| `201 Created` | POST created a resource (`Location` header) |
| `204 No Content` | Success, no body (DELETE) |
| `400 Bad Request` | Malformed request |
| `401 Unauthorized` | Not authenticated |
| `403 Forbidden` | Authenticated but not allowed |
| `404 Not Found` | Resource missing |
| `409 Conflict` | Duplicate / version conflict |
| `422 Unprocessable Entity` | Validation failed (Laravel default) |
| `429 Too Many Requests` | Rate limited |
| `500 Internal Server Error` | Unhandled exception |

> [!TIP]
> **One-liner:** Status codes are grouped 1xx–5xx. I pick a specific 2xx/4xx/5xx so the client can branch without parsing a message string.

---

## Q3. What are the general types of APIs?

**Answer:**

“API” just means a contract for programs to talk. The types I name in interviews:

| Type | Style | Typical transport |
|------|--------|-------------------|
| **REST** | Resources + HTTP verbs + status codes | HTTP + JSON |
| **RPC** | Call a procedure (`createOrder`) | HTTP, gRPC |
| **GraphQL** | Client asks for the exact fields | HTTP + one `/graphql` endpoint |
| **SOAP** | XML envelopes, WSDL contracts | HTTP + XML |
| **WebSocket / SSE** | Push / streaming, not request-response only | WS / HTTP |

Also useful split: **public HTTP APIs** vs **library APIs** (a PHP package’s classes). Interviewers almost always mean HTTP.

I default to REST for CRUD backends. GraphQL when many clients need different shapes. gRPC for internal service-to-service with strict contracts. SOAP when an enterprise partner still requires it.

> [!TIP]
> **One-liner:** Common API styles are REST, RPC/gRPC, GraphQL, SOAP, and realtime (WebSocket). REST over HTTP + JSON is the default for Laravel backends.

---

## Q4. What is a RESTful API, and how does it work? Which protocol does it rely on, and how does it utilize HTTP features (e.g., headers and status codes)?

**Answer:**

A RESTful API exposes **resources** (nouns) over **HTTP**. The client is stateless: every request carries what the server needs (auth header, IDs). The server does not remember the previous click.

It relies on **HTTP** (almost always HTTPS in production). REST is an architectural style, not a new protocol.

**How it uses HTTP:**

| HTTP feature | REST use |
|--------------|----------|
| URL | Resource identity: `/api/orders/15` |
| Method | Operation: GET read, POST create, PATCH update, DELETE |
| Status code | Outcome: 201 created, 404 missing, 422 invalid |
| `Content-Type` / `Accept` | JSON in/out (`application/json`) |
| `Authorization` | Bearer token / Sanctum |
| `Location` | URI of a newly created resource |
| `ETag` / `If-Match` | Optional optimistic concurrency |
| Query string | Filtering, pagination: `?page=2&status=paid` |

```http
POST /api/orders HTTP/1.1
Host: api.example.com
Authorization: Bearer 123
Accept: application/json
Content-Type: application/json

{"product_id": 9, "qty": 2}
```

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/orders/15

{"id": 15, "status": "pending"}
```

Laravel pieces: `routes/api.php`, Form Requests, API Resources (shape JSON), Sanctum, throttle middleware. I keep URLs as nouns, verbs in the HTTP method — not `POST /api/getOrder`.

> [!TIP]
> **One-liner:** REST uses HTTP as the protocol: URLs name resources, methods name actions, headers carry metadata/auth, status codes carry the result, JSON is the usual representation.

---

## Q5. How do you approach API design, and what tools do you use?

**Answer:**

I design the **contract first**, then implement it.

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

> [!TIP]
> **One-liner:** I design resources and error codes first, document them (OpenAPI/Postman), then implement with Laravel Form Requests, API Resources, and token auth.

---

## Q6. What is the difference between PUT and POST?

**Answer:**

| Aspect | `POST` | `PUT` |
|--------|--------|-------|
| Typical intent | **Create** (or a non-CRUD action) | **Replace** the resource at a known URI |
| URI | Often the collection: `/api/users` | The resource: `/api/users/15` |
| Idempotent | No — two POSTs can create two rows | Yes — repeating the same PUT yields the same state |
| Who assigns the id | Server | Client already knows the URI (or sends the full replacement) |
| Body | New resource fields | **Complete** representation |

```http
POST /api/articles          → 201  created (new id)
PUT  /api/articles/42       → 200  replaced article 42 entirely
```

`PATCH` is the sibling people mix in: **partial** update, not a full replace. If the client sends only `{ "title": "New" }`, that is PATCH, not PUT.

> [!TIP]
> **One-liner:** POST creates (not idempotent, server assigns the URI). PUT replaces an existing resource at a known URI and is idempotent.

---

## Q7. When might you use POST for an update operation?

**Answer:**

REST textbooks say PATCH/PUT for updates. In real systems I still use POST when HTTP or the client cannot do the “pure” verb well.

**Legitimate cases:**

- **HTML forms** only submit GET/POST. Laravel’s `_method=PUT` (method spoofing) is still a POST on the wire.
- **File uploads** — PHP does not fill `$_FILES` for PUT/PATCH. Multipart updates are usually **POST** (often with method spoofing). See Q8.
- **Non-idempotent “actions”** — `POST /api/orders/15/cancel`, `POST /api/invoices/15/retry-payment`. These are verbs-as-resources, not field patches.
- **Partial processing that is not a simple replace** — start a job, apply a coupon, the result may differ if sent twice (you then add idempotency keys).
- **Legacy clients / some proxies** that block PUT/PATCH.

I do **not** use POST for updates just because “it works”. If the client is a JSON API with no files, I prefer `PATCH /api/orders/15`.

> [!TIP]
> **One-liner:** POST for updates when the client is a form, when uploading files (PHP multipart), or when the operation is an action (`/cancel`) rather than a field replace.

---

## Q8. Suppose you have a form containing text data, images, and a PDF file, and you need to create an API endpoint for an update operation. Address the following:
- What are the appropriate status codes to return?
- Which HTTP request method should be used?
- What types of headers and body structure are required?
- How should the data be formatted in the request (e.g., multipart/form-data)?

**Answer:**

This is a **multipart update with files**. In PHP/Laravel I treat that as **POST** (optionally spoofing PUT/PATCH), not a raw PUT JSON body.

### Method

Use **`POST`** to the resource (e.g. `/api/profiles/1`).

Reason: PHP populates `$_FILES` / `$request->file()` for POST multipart. PUT/PATCH multipart is unreliable in PHP. If I want REST naming, I still POST and add `_method=PATCH` (web) or document “POST = update with files” for the API.

### Body format

`multipart/form-data` — **not** `application/json`. JSON cannot carry raw file bytes without base64 (worse for large PDFs/images).

```http
POST /api/profiles/1 HTTP/1.1
Authorization: Bearer 123
Accept: application/json
Content-Type: multipart/form-data; boundary=----Boundary

------Boundary
Content-Disposition: form-data; name="bio"

Backend developer
------Boundary
Content-Disposition: form-data; name="avatar"; filename="photo.jpg"
Content-Type: image/jpeg

<binary>
------Boundary
Content-Disposition: form-data; name="cv"; filename="cv.pdf"
Content-Type: application/pdf

<binary>
------Boundary--
```

The browser/Postman sets `Content-Type` **including the boundary**. I never set `multipart/form-data` by hand without a boundary.

In Laravel:

```php
$request->validate([
    'bio' => ['required', 'string'],
    'avatar' => ['nullable', 'image', 'max:2048'],
    'cv' => ['nullable', 'file', 'mimes:pdf', 'max:5120'],
]);

$path = $request->file('avatar')?->store('avatars');
```

### Headers

| Header | Value |
|--------|--------|
| `Authorization` | `Bearer {token}` (or Sanctum cookie for SPA) |
| `Accept` | `application/json` |
| `Content-Type` | `multipart/form-data; boundary=…` (client-generated) |

Do not send `application/json` at the same time as the files.

### Status codes

| Code | When |
|------|------|
| `200 OK` | Updated; return the resource JSON |
| `204 No Content` | Updated; no body (less common if the client needs new file URLs) |
| `401` / `403` | Missing/invalid auth or not the owner |
| `404` | Profile id does not exist |
| `413` | File too large (server/php.ini) |
| `422` | Validation (bad mime, missing `bio`) |
| `500` | Storage/unexpected failure |

I store files on disk/S3, save paths on the model, and return JSON with public URLs — I do not echo the PDF back in the same response unless asked.

> [!TIP]
> **One-liner:** POST + `multipart/form-data` for text + files (PHP’s file upload model), `Accept: application/json`, and 200/422/401/403/404 as the main status codes.

---

## Q9. How do you integrate with an external API?

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

---

> [!NOTE]
> Additional reference for API questions: [roadmap.sh/questions/rest-api](https://roadmap.sh/questions/rest-api)
