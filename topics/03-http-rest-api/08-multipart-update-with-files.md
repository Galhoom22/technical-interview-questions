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

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| `multipart/form-data` | Body format that can carry files + text fields. |
| `$_FILES` | PHP’s file-upload array. Filled on POST multipart, not on PUT. |
| Boundary | Marker in `Content-Type` that splits each part. The client sets it. |
| `422` | Validation failed. RFC/MDN: Unprocessable **Content**. Laravel docs still say Unprocessable Entity. |

### 🧠 Analogy

JSON cannot carry a PDF cleanly. `multipart/form-data` is the **envelope with pockets**: one pocket for bio, one for the photo, one for the CV. PHP only unpacks those pockets on POST.

### Method

Use **`POST`** to the resource (e.g. `/api/profiles/1`).

Reason: PHP populates `$_FILES` / `$request->file()` for POST multipart. PUT/PATCH multipart is unreliable in PHP. If I want REST naming, I still POST and add `_method=PATCH` (web) or document “POST = update with files” for the API.

### Body format

`multipart/form-data` — **not** `application/json`. JSON cannot carry raw file bytes without base64 (worse for large PDFs/images).

---

### 📘 Official ([PHP: POST method uploads](https://www.php.net/manual/en/features.file-upload.post-method.php) + [MDN: MIME types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/MIME_types))

```html
<form enctype="multipart/form-data" action="__URL__" method="POST">
    <input type="hidden" name="MAX_FILE_SIZE" value="30000" />
    Send this file: <input name="userfile" type="file" />
</form>
```

`Content-Type` is `multipart/form-data`. PHP fills `$_FILES` for POST, not for PUT.

### 💼 In production

```php
public function update(UpdateProfileRequest $request, Profile $profile): JsonResponse
{
    $data = $request->validated();

    if ($request->hasFile('avatar')) {
        $data['avatar_path'] = $request->file('avatar')->store('avatars');
    }

    if ($request->hasFile('cv')) {
        $data['cv_path'] = $request->file('cv')->store('cvs');
    }

    $profile->update($data);

    return ProfileResource::make($profile)
        ->response()
        ->setStatusCode(200);
}
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

---

### ⚠️ Watch out

> [!WARNING]
> Never set `Content-Type: multipart/form-data` by hand without the boundary. Do not send JSON and files as one JSON body (base64 PDFs hurt). PUT will not fill `$_FILES`.

### 💬 If they follow up

> [!NOTE]
> - Status? 200 + JSON (new URLs), 422 validation, 413 too large.
> - Where do files live? Disk/S3 path on the model — not the binary in the JSON.

---

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
