# What is the difference between PUT and POST?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

`POST` typically **creates**. `PUT` **replaces** the whole resource at a known URI.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| POST | Create, or process this entity. Not idempotent. |
| PUT | Replace the whole resource at a known URI. Idempotent. |
| PATCH | Partial update — only the fields you send. |
| Idempotent | Repeating the same request leaves the same state. |
| Resource | A noun the API exposes (`/orders/15`). |

### 🧠 Analogy

POST is **dropping a new box on the counter** (the shop assigns the ticket number). PUT is **replacing the box already on shelf 15** — same shelf, whole new contents. Twice PUT, still one box.

| Aspect | `POST` | `PUT` |
|--------|--------|-------|
| Typical intent | **Create** (or a non-CRUD action) | **Replace** the resource at a known URI |
| URI | Often the collection: `/api/users` | The resource: `/api/users/15` |
| Idempotent | No — two POSTs can create two rows | Yes — repeating the same PUT yields the same state |
| Who assigns the id | Server | Client already knows the URI (or sends the full replacement) |
| Body | New resource fields | **Complete** representation |

---

### 📘 Official ([MDN: POST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/POST) + [PUT](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/PUT))

```http
POST /test HTTP/1.1
Content-Type: application/json

{"name": "New"}

PUT /new.html HTTP/1.1
Content-Type: text/html

<full replacement of new.html>
```

POST: process this entity (often create). PUT: store this as the resource at this URI (idempotent replace).

### 💼 In production

```http
POST /api/orders              → 201  new order, server assigns id
PUT  /api/orders/15           → 200  replace the whole order document
PATCH /api/orders/15          → 200  only { "status": "paid" }
```

`PATCH` is the sibling people mix in: **partial** update, not a full replace. If the client sends only `{ "title": "New" }`, that is PATCH, not PUT.

---

### ⚠️ Watch out

> [!WARNING]
> Sending one field with PUT is a lie — that is PATCH. Two identical POSTs can mean two rows.

### 💬 If they follow up

> [!NOTE]
> - Who assigns the id? POST: server. PUT: the URI already exists.
> - Idempotent? PUT yes, POST no.

---

> [!TIP]
> **One-liner:** POST creates (not idempotent, server assigns the URI). PUT replaces an existing resource at a known URI and is idempotent.

**Source:** [MDN: POST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/POST) and [MDN: PUT](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/PUT) — official semantics this table follows.

**Learn more:**
- [MDN: PATCH](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/PATCH) — partial update (the third verb interviewers mix in)
- [MDN: Idempotent](https://developer.mozilla.org/en-US/docs/Glossary/Idempotent) — why repeating PUT is safe
- [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) — POST vs PUT in the spec

---

[← Previous](./02-status-codes.md) · [Topic](./README.md) · [Next →](./04-post-for-update.md)