# When might you use POST for an update operation?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

REST textbooks say PATCH/PUT for updates. In real systems I still use POST when HTTP or the client cannot do the “pure” verb well.

**🔑 Key terms:**

| Term | Plain meaning |
|------|----------------|
| Method spoofing | HTML form still POSTs; Laravel reads `_method=PUT`. |
| Action URL | A verb as a resource (`POST /orders/15/cancel`) — not a field patch. |
| `$_FILES` | PHP’s file-upload array. Filled on POST multipart, not on PUT. |
| PATCH | Partial update — only the fields you send. |

**🧠 Analogy:**

The textbook wants PATCH. Real browsers only POST forms. Real PHP only fills `$_FILES` on POST. So the wire is often POST, with `_method=PATCH` or an action URL like `/cancel`.

**Legitimate cases:**

- **HTML forms** only submit GET/POST. Laravel’s `_method=PUT` (method spoofing) is still a POST on the wire.
- **File uploads** — PHP does not fill `$_FILES` for PUT/PATCH. Multipart updates are usually **POST** (often with method spoofing). See Q8.
- **Non-idempotent “actions”** — `POST /api/orders/15/cancel`, `POST /api/invoices/15/retry-payment`. These are verbs-as-resources, not field patches.
- **Partial processing that is not a simple replace** — start a job, apply a coupon, the result may differ if sent twice (you then add idempotency keys).
- **Legacy clients / some proxies** that block PUT/PATCH.

I do **not** use POST for updates just because “it works”. If the client is a JSON API with no files, I prefer `PATCH /api/orders/15`.

**📘 Official** ([Laravel: Form Method Spoofing](https://laravel.com/docs/13.x/routing#form-method-spoofing)):

```html
<form action="/foo/bar" method="POST">
    @csrf
    @method('PUT')
</form>
```

The browser still sends POST. Laravel reads `_method=PUT`.

**💼 In production:**

```http
POST /api/orders/15/cancel HTTP/1.1
Authorization: Bearer {token}

POST /account/profile HTTP/1.1
Content-Type: multipart/form-data; boundary=...
# HTML profile form with avatar — PHP $_FILES only on POST
```

**⚠️ Watch out:**

Do not use POST for JSON updates “because it works.” If the client can PATCH and there are no files, PATCH.

**💬 If they follow up:**

- Method spoofing? `@method('PUT')` — still POST on the wire.
- `/cancel`? Action resource — POST is the right verb.

> [!TIP]
> **One-liner:** POST for updates when the client is a form, when uploading files (PHP multipart), or when the operation is an action (`/cancel`) rather than a field replace.

**Source:** [Laravel: Form Method Spoofing](https://laravel.com/docs/13.x/routing#form-method-spoofing) — HTML forms only GET/POST, so updates often travel as POST + `_method`. File uploads: [PHP: POST method uploads](https://www.php.net/manual/en/features.file-upload.post-method.php).

**Learn more:**
- [PHP: PUT method support](https://www.php.net/manual/en/features.file-upload.put-method.php) — PUT files come on `php://input`, not `$_FILES`
- [MDN: POST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/POST) — POST is allowed to mean “process this”, not only “create”
- [MDN: PATCH](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/PATCH) — prefer this for JSON partial updates when the client can send it

---

[← Previous](./06-put-vs-post.md) · [Topic](./README.md) · [Next →](./08-multipart-update-with-files.md)
