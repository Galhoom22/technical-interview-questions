# What is the difference between a Seeder and a Factory?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

They work together. They are not the same layer.

| | Factory | Seeder |
|---|----------|--------|
| Role | **How one model looks** when fake | **What gets inserted**, and how many |
| Runs | From tests or from a seeder | `php artisan db:seed` |
| Knows about | One model (and related factories) | Whole database / feature set |
| Example | `UserFactory` definition + `admin()` state | “Create 3 roles, 1 admin, 50 users” |

**Official** ([Laravel: Eloquent Factories](https://laravel.com/docs/13.x/eloquent-factories) + [Seeding](https://laravel.com/docs/13.x/seeding)):

```php
User::factory()->count(3)->create(); // factory: how one User looks

public function run(): void          // seeder: what to insert
{
    User::factory(10)->create();
}
```

**In production:**

```php
it('applies a coupon', function () {
    $order = Order::factory()->create(['total_cents' => 5000]);
    // test uses the factory only — no DatabaseSeeder
});

public function run(): void
{
    Coupon::query()->updateOrCreate(
        ['code' => 'WELCOME10'],
        ['percent' => 10, 'active' => true],
    );

    if (! app()->isProduction()) {
        Order::factory()->count(20)->create();
    }
}
```

Tests usually call **factories directly** (fast, isolated). Seeders are for **environments** (local demo, staging). Production rarely runs dummy factories; it may run a seeder for required lookup rows only.

> [!TIP]
> **One-liner:** A factory describes one fake model. A seeder decides which records to insert. Seeders often call factories.

**Source:** [Laravel: Eloquent Factories](https://laravel.com/docs/13.x/eloquent-factories) (the recipe) and [Seeding](https://laravel.com/docs/13.x/seeding) (the meal plan) — official split this table follows.

**Learn more:**
- [Laravel: HTTP Tests](https://laravel.com/docs/13.x/http-tests) — tests usually call factories directly, not full seeders
- [Laravel: Database Testing](https://laravel.com/docs/13.x/database-testing) — `RefreshDatabase`, seeders in test setup
- [Laravel: Migrations](https://laravel.com/docs/13.x/migrations) — neither factories nor seeders replace schema versioning

---

[← Previous](./09-seeders.md) · [Topic](./README.md)
