# How do you design an update API that accepts text, images, and a PDF?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

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
| `422` | Validation (bad mime, missing `bio`). RFC 9110 name: Unprocessable Content. |
| `500` | Storage/unexpected failure |

I store files on disk/S3, save paths on the model, and return JSON with public URLs — I do not echo the PDF back in the same response unless asked.

> [!TIP]
> **One-liner:** POST + `multipart/form-data` for text + files (PHP’s file upload model), `Accept: application/json`, and 200/422/401/403/404 as the main status codes.

**Source:** [PHP: POST method uploads](https://www.php.net/manual/en/features.file-upload.post-method.php) — `enctype="multipart/form-data"` and `$_FILES`; plus [MDN: MIME types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/MIME_types) for `multipart/form-data`.

**Learn more:**
- [PHP: PUT method support](https://www.php.net/manual/en/features.file-upload.put-method.php) — why PUT does not fill `$_FILES` (read `php://input` instead)
- [Laravel: File Storage](https://laravel.com/docs/13.x/filesystem) — `$request->file()`, `store()`, S3
- [Laravel: Validation](https://laravel.com/docs/13.x/validation#validating-files) — `image`, `mimes:pdf`, `max:`
- [MDN: 413 Content Too Large](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/413) — file bigger than server limits

---

[← Previous](./07-post-for-update.md) · [Topic](./README.md) · [Next →](./09-external-api-integration.md)
