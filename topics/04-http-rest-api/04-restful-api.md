# What is a RESTful API, and how does it work? Which protocol does it rely on, and how does it utilize HTTP features (e.g., headers and status codes)?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

A RESTful API exposes **resources** (nouns) over **HTTP**. The client is stateless: every request carries what the server needs (auth header, IDs). The server does not remember the previous click.

It relies on **HTTP** (almost always HTTPS in production). REST is an architectural style, not a new protocol.

**Key terms:**

| Term | Plain meaning |
|------|----------------|
| REST | Architectural style — resources over HTTP, stateless. Not a new protocol. |
| Resource | A noun the API exposes (`/orders/15`). |
| Stateless | Every request carries what the server needs. No “previous click” on the server. |
| Representation | The JSON (or HTML) body that stands for the resource. |
| Uniform interface | URLs name resources, methods name actions, status codes name outcomes. |

**Analogy:**

REST is a **stateless post office**: every letter carries the address and the stamp. The clerk does not remember your last visit. HTTPS is the truck; REST is how you write the envelope.

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

**Official** ([MDN: REST](https://developer.mozilla.org/en-US/docs/Glossary/REST) + [HTTP methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)):

```http
GET /items/2 HTTP/1.1
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json

{"id": 2, "name": "Example"}
```

**In production:**

```http
POST /api/orders HTTP/1.1
Authorization: Bearer {token}
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

**Watch out:**

`POST /api/getOrder` is RPC dressed as HTTP. REST is not a new protocol — it *uses* HTTP.

**If they follow up:**

- Auth header? `Authorization: Bearer`. Create? `201` + `Location`.
- Session on the API? Not for stateless Bearer; Sanctum SPA still uses cookies + CSRF.

> [!TIP]
> **One-liner:** REST uses HTTP as the protocol: URLs name resources, methods name actions, headers carry metadata/auth, status codes carry the result, JSON is the usual representation.

**Source:** [MDN: REST](https://developer.mozilla.org/en-US/docs/Glossary/REST) plus [HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods) and [status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status) — REST is an HTTP style, not a new protocol.

**Learn more:**
- [Fielding: REST architectural style](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) — constraints (stateless, uniform interface, …)
- [Laravel: Eloquent API Resources](https://laravel.com/docs/13.x/eloquent-resources) — shaping the JSON representation
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html) — how you document that HTTP contract
- [JSON:API](https://jsonapi.org/format/) — one optional resource+relationship convention (not required for “REST”)

---

[← Previous](./03-api-types.md) · [Topic](./README.md) · [Next →](./05-api-design.md)
