# What are the different HTTP methods?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

HTTP methods (verbs) tell the server **what kind of operation** the client wants on a resource.

**🔑 Key terms:**

| Term | Plain meaning |
|------|----------------|
| HTTP method | The verb — what kind of operation (GET, POST, PUT, PATCH, DELETE). |
| Resource | A noun the API exposes (`/orders/15`). |
| Safe | Must not change server state (GET, HEAD, OPTIONS). |
| Idempotent | Repeating the same request leaves the same state. |

**🧠 Analogy:**

HTTP methods are **verbs on a noun**. GET is reading the order; POST is creating one; DELETE is throwing it away. Safe means “reading the sign”; idempotent means “pressing the same button twice doesn’t make two orders.”

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

**📘 Official** ([MDN: HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)):

```http
GET /index.html HTTP/1.1
POST /test HTTP/1.1
PUT /new.html HTTP/1.1
DELETE /file.html HTTP/1.1
```

**💼 In production:**

```http
GET    /api/orders
POST   /api/orders
GET    /api/orders/15
PATCH  /api/orders/15
DELETE /api/orders/15
POST   /api/orders/15/cancel
```

**⚠️ Watch out:**

GET must not change state. Two POSTs can create two rows. PUT replaces the **whole** document — a partial body is PATCH.

**💬 If they follow up:**

- OPTIONS? CORS preflight / “what methods are allowed.”
- POST `/cancel`? An action, not a field patch — legitimate.

> [!TIP]
> **One-liner:** The verbs you use every day are GET (read), POST (create/action), PUT (full replace), PATCH (partial update), DELETE (remove), plus HEAD and OPTIONS.

**Source:** [MDN: HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods) — official method list plus the safe / idempotent table this answer uses.

**Learn more:**
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) — the actual HTTP spec (methods, idempotency)
- [MDN: Safe methods](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP) — why GET must not change state
- [MDN: Idempotent](https://developer.mozilla.org/en-US/docs/Glossary/Idempotent) — why PUT/DELETE can be retried

---

[Topic](./README.md) · [Next →](./02-status-codes.md)
