# ♻️ Refactoring & Best Practices

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

## Q1. If you encounter a massive method (e.g., 400 lines of code), how would you approach refactoring it?

**Answer:**

I do **not** rewrite the 400 lines in one heroic commit. I make it safe, understand it, then peel it into pieces that each do one thing.

**1. Freeze behavior**

- If tests exist, run them. If not, I add **characterization tests** around the current output (same input → same result) before changing structure.
- Note side effects: emails, payments, `DB::transaction`, deletes. Those are the landmines.

**2. Read for seams, not for style**

A 400-line method is usually several *jobs* glued together. I label them in comments first:

```
validate → load models → calculate prices → persist → notify → build response
```

**3. Small mechanical steps** (each step still green)

| Smell | Move |
|-------|------|
| Nested `if`/`foreach` | Early return; extract method |
| Repeated blocks | Private method or new class |
| SQL / HTTP in the middle | Query/service object |
| Price/tax rules | Domain class (`Totals`, `TaxCalculator`) |
| Controller soup | Form Request + Action/Service |
| God object | Split by responsibility (SRP) |

```php
// before: one method does everything
public function checkout(Request $request) { /* 400 lines */ }

// after: orchestration only
public function checkout(CheckoutRequest $request): JsonResponse
{
    $cart = $this->carts->forUser($request->user());
    $totals = $this->totals->for($cart, $request->coupon());
    $order = $this->orders->place($cart, $totals);

    return OrderResource::make($order);
}
```

**4. Rules I keep**

- One PR / one concern when possible: “extract pricing” separately from “extract notifications”.
- Rename once the name is honest. Bad names hide the next split.
- Stop when the method *orchestrates* instead of *implements* — 400 lines becomes a 20-line story plus named collaborators.
- I do not “clean” and change business rules in the same diff.

**5. What I would say if they ask “how long?”**

First pass: tests + extract 3–5 methods. Next: push a cluster into a class. I never promise a full rewrite by Friday unless the tests already lock the behavior.

> [!TIP]
> **One-liner:** Characterize with tests, find the separate jobs inside the method, extract them one green step at a time, and keep the original method as a short orchestrator.

**Source:** [Extract Function](https://refactoring.com/catalog/extractFunction.html) — Fowler’s catalog (alias: Extract Method); this is the first move on a 400-line method.

**Learn more:**
- [Refactoring catalog](https://refactoring.com/catalog/) — Extract Class, Replace Temp with Query, and the rest of the catalog
- [Laravel: Testing](https://laravel.com/docs/13.x/testing) — characterization tests are still tests; lock behavior with Pest/PHPUnit before you extract
- [Characterization test](https://en.wikipedia.org/wiki/Characterization_test) — the term (Michael Feathers); Wikipedia summary, not a spec
- [Laravel: Controllers](https://laravel.com/docs/13.x/controllers) — keep HTTP thin; push work into actions/services
- [Laravel: Form Request Validation](https://laravel.com/docs/13.x/validation#form-request-validation) — first extraction from a fat controller is often the Form Request
