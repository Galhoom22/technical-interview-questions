# What are the different HTTP methods?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

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

**Source:** [MDN: HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods) — official method list plus the safe / idempotent table this answer uses.

**Learn more:**
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html) — the actual HTTP spec (methods, idempotency)
- [MDN: Safe methods](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP) — why GET must not change state
- [MDN: Idempotent](https://developer.mozilla.org/en-US/docs/Glossary/Idempotent) — why PUT/DELETE can be retried

---

[Topic](./README.md) · [Next →](./02-status-codes.md)
