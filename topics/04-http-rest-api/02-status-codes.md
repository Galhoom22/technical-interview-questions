# What are HTTP status codes, and what are their main categories?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Status codes are a **3-digit signal** of how the request turned out. The first digit is the category; the body (if any) carries details.

**Key terms:**

| Term | Plain meaning |
|------|----------------|
| Status code | A 3-digit signal of how the request turned out. |
| `2xx` | Success. |
| `4xx` | The **client** did something wrong (auth, validation, missing id). |
| `5xx` | The **server** failed. |
| `422` | Validation failed. RFC/MDN: Unprocessable **Content**. Laravel docs still say Unprocessable Entity. |

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
| `422 Unprocessable Content` | Validation failed (Laravel APIs). RFC 9110 / MDN name is **Unprocessable Content**; Laravel’s validation docs still say **Unprocessable Entity**. Same status code `422`. |
| `429 Too Many Requests` | Rate limited |
| `500 Internal Server Error` | Unhandled exception |

**Official** ([MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)):

```http
HTTP/1.1 201 Created
Location: /new.html

HTTP/1.1 404 Not Found

HTTP/1.1 422 Unprocessable Content
```

**In production:**

```php
return OrderResource::make($order)
    ->response()
    ->setStatusCode(201)
    ->header('Location', route('orders.show', $order));

// validation failure → 422 JSON errors (Laravel)
// unknown order → abort(404)
// someone else's order → abort(403)
```

> [!TIP]
> **One-liner:** Status codes are grouped 1xx–5xx. I pick a specific 2xx/4xx/5xx so the client can branch without parsing a message string.

**Source:** [MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status) — 1xx–5xx groups and the codes listed in this answer.

**Learn more:**
- [RFC 9110 — Status Codes](https://www.rfc-editor.org/rfc/rfc9110.html#name-status-codes) — normative definitions
- [MDN: 422 Unprocessable Content](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/422) — RFC 9110 name for this code
- [Laravel: Validation](https://laravel.com/docs/13.x/validation) — JSON validation failures return `422`
- [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html) — which codes not to leak internals with

---

[← Previous](./01-http-methods.md) · [Topic](./README.md) · [Next →](./03-api-types.md)
