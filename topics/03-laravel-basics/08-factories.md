# What are Factories?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

Factories define **how to build fake (or realistic) model data**. They power tests and seeders so you never insert raw arrays by hand.

```php
class UserFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'password' => 'password', // hashed by the model's `hashed` cast
        ];
    }

    public function admin(): static
    {
        return $this->state(['is_admin' => true]);
    }
}

User::factory()->count(10)->create();
User::factory()->admin()->create();
User::factory()->make(); // in memory, no INSERT
```

`create()` persists. `make()` does not. States (`admin()`) and sequences customize subsets. `has(Post::factory()->count(3))` builds relationships.

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
