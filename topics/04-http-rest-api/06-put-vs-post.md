# What is the difference between PUT and POST?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

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

**Source:** [MDN: POST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/POST) and [MDN: PUT](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/PUT) — official semantics this table follows.

**Learn more:**
- [MDN: PATCH](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/PATCH) — partial update (the third verb interviewers mix in)
- [MDN: Idempotent](https://developer.mozilla.org/en-US/docs/Glossary/Idempotent) — why repeating PUT is safe
- [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html) — POST vs PUT in the spec

---

[← Previous](./05-api-design.md) · [Topic](./README.md) · [Next →](./07-post-for-update.md)
