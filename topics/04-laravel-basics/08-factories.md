# What are Factories?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Factories define **how to build fake (or realistic) model data**. They power tests and seeders so you never insert raw arrays by hand.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Factory | A blueprint for fake (or realistic) model attributes. |
| `create()` | Build **and** `INSERT` the model. |
| `make()` | Build in memory. No `INSERT`. |
| State | A named variation on a factory (`pending()`, `admin()`). |
| `fake()` | Laravel helper around Faker (`fake()->email()`). |

### 🧠 Analogy

A factory is the **cookie recipe**: how one `User` or `Order` looks when fake. `create()` bakes and puts it in the tin; `make()` only mixes the dough.

---

### 📘 Official ([Laravel: Eloquent Factories](https://laravel.com/docs/13.x/eloquent-factories))

```php
class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'password' => 'password',
        ];
    }
}

User::factory()->count(3)->make();
User::factory()->count(3)->create();
```

### 💼 In production

```php
class OrderFactory extends Factory
{
    public function definition(): array
    {
        return [
            'user_id' => User::factory(),
            'status' => 'paid',
            'total_cents' => 4999,
            'currency' => 'USD',
        ];
    }

    public function pending(): static
    {
        return $this->state(['status' => 'pending']);
    }
}

it('lists the user orders', function () {
    $user = User::factory()->create();
    Order::factory()->count(3)->for($user)->create();
    Order::factory()->pending()->for($user)->create();

    $this->actingAs($user)->getJson('/api/orders')->assertOk();
});
```

`create()` persists. `make()` does not. States (`admin()`) and sequences customize subsets. `has(Post::factory()->count(3))` builds relationships.

---

### ⚠️ Watch out

> [!WARNING]
> Do not insert raw arrays in tests. Hashing: the `hashed` cast hashes `password` when set — do not double-hash unless you know you need to.

### 💬 If they follow up

> [!NOTE]
> - Relationships in factories? `has()` / `for()`.
> - States? `Order::factory()->pending()->create()`.

---

> [!TIP]
> **One-liner:** A factory is a blueprint for fake model attributes — used in tests and seeders via `Model::factory()->create()`.

**Source:** [Laravel: Eloquent Factories](https://laravel.com/docs/13.x/eloquent-factories) — `definition()`, states, `create()` vs `make()`, relationships.

**Learn more:**
- [Laravel: Seeding](https://laravel.com/docs/13.x/seeding) — where factories get called in bulk
- [Laravel: HTTP Tests](https://laravel.com/docs/13.x/http-tests) — factories inside feature tests
- [Laravel: Helpers — `fake()`](https://laravel.com/docs/13.x/helpers#method-fake) — Faker through Laravel’s `fake()` helper
- [Laravel: Hashing](https://laravel.com/docs/13.x/hashing) — the `hashed` cast hashes `password` when it is set

---

[← Previous](./07-eloquent-vs-query-builder.md) · [Topic](./README.md) · [Next →](./09-seeders.md)