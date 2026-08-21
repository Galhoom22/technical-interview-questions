# Explain your process for testing APIs using an API client like Postman.

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)

**Answer:**

I treat Postman as a **repeatable contract check**, not a one-off “send” button. Same collection works for me, the frontend, and CI (Newman).

**Key terms:**

| Term | Plain meaning |
|------|----------------|
| Collection | A folder of Postman requests for one API. |
| Environment | `local` / `staging` variables (`baseUrl`, `token`). |
| `pm.test` | A Postman assertion on the response. |
| Newman | CLI that runs the same collection in CI. |
| Contract | The URLs, methods, JSON shapes, and status codes clients can depend on. |

**Analogy:**

Postman is a **shared checklist**, not a screenshot of one 200. Login once, stamp the token, then the folder proves 201/401/422 the same way every time.

**Setup:**

1. **Collection** per API (`Orders API`).
2. **Environments** — `local`, `staging`, `production` (prod is read-only GETs if we use it at all).
3. **Variables** — `baseUrl`, `token`, `userId` so requests are not hardcoded.
4. **Auth once** — a login request stores `token` with `pm.environment.set('token', ...)` for the rest of the folder.

**Per endpoint I always check:**

| Layer | What I verify |
|-------|----------------|
| Status | 200 / 201 / 204 on the happy path |
| Headers | `Content-Type: application/json` |
| Body | Shape, types, ids, pagination keys |
| Auth | No token → `401`; wrong user → `403` |
| Validation | Bad payload → `422` + field errors |
| Missing | Unknown id → `404` |

**Official** ([Postman: Write scripts to test API response data](https://learning.postman.com/docs/tests-and-scripts/write-scripts/test-scripts)):

```javascript
pm.test('Status code is 200', function () {
    pm.response.to.have.status(200);
});
```

**In production:**

```javascript
pm.test('creates order', function () {
    pm.response.to.have.status(201);
    const json = pm.response.json();
    pm.expect(json).to.have.property('id');
    pm.expect(json.status).to.eql('pending');
    pm.environment.set('orderId', json.id);
});

pm.test('rejects unauthenticated checkout', function () {
    pm.response.to.have.status(401);
});
```

**Flow I actually run:**

1. Auth → save token.
2. Happy path CRUD in order (create → show → update → list → delete), chaining ids through variables.
3. Negative cases (401, 403, 422, 404) in the same folder.
4. Edge cases: pagination, filters, empty lists, file upload (`form-data`) if the endpoint takes files.
5. Export the collection. Optional: Newman in CI against staging.

This does **not** replace Pest/PHPUnit. Postman is for the HTTP contract and for sharing with non-PHP teammates. Automated tests in the repo still own regressions (`php artisan test`).

**Watch out:**

Hardcoded URLs and tokens die. Prod collections should be read-only GETs if you use them at all. Postman is not a substitute for `php artisan test`.

**If they follow up:**

- CI? Newman / Postman CLI on the same collection.
- Negative cases? 401, 403, 422, 404 in the same folder as the happy path.

> [!TIP]
> **One-liner:** Environments + collection variables, login once, then assert status, JSON shape, and 401/422/404 — save the collection so it is repeatable, not a screenshot of one 200.

**Source:** [Postman: Write scripts to test API response data](https://learning.postman.com/docs/tests-and-scripts/write-scripts/test-scripts) — `pm.test`, `pm.response`, Chai assertions (the process this answer describes).

**Learn more:**
- [Postman: Variables](https://learning.postman.com/docs/sending-requests/variables/variables/) — `baseUrl`, `token`, chaining ids
- [Collection Runner](https://learning.postman.com/docs/tests-and-scripts/running-collections/intro-to-collection-runs) — run the whole folder, not one request
- [Postman CLI](https://learning.postman.com/docs/postman-cli/postman-cli-overview/) — same collection in CI (Newman is the older CLI; Postman now documents this)
- [Laravel: Testing](https://laravel.com/docs/13.x/testing) — Pest and PHPUnit ship with Laravel 13; `php artisan test`
- [Laravel: HTTP Tests](https://laravel.com/docs/13.x/http-tests) — repo tests that must still exist beside Postman

---

[Topic](./README.md)
