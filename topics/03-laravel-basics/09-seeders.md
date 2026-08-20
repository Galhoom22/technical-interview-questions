# What is a Seeder?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

A seeder is a class that **inserts data** into the database on purpose: lookup tables, an admin user, or a full demo dataset.

```php
class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            RoleSeeder::class,
            AdminUserSeeder::class,
        ]);

        User::factory()->count(50)->create();
    }
}
```

```bash
php artisan db:seed
php artisan migrate:fresh --seed
```

Seeders are for **what to insert and how much**. They often **call factories** for dummy rows, and insert fixed rows (countries, roles) without factories.

> [!TIP]
> **One-liner:** A seeder is a script that fills the database — roles, admin users, demo data — usually run with `db:seed`.

**Source:** [Laravel: Seeding](https://laravel.com/docs/13.x/seeding) — `DatabaseSeeder`, `call()`, `db:seed`, `migrate:fresh --seed`.

**Learn more:**
- [Laravel: Eloquent Factories](https://laravel.com/docs/13.x/eloquent-factories) — dummy rows inside a seeder
- [Laravel: Migrations](https://laravel.com/docs/13.x/migrations) — schema first, data second
- [Laravel: Database Testing](https://laravel.com/docs/13.x/database-testing) — `RefreshDatabase` vs running seeders in tests

---

[← Previous](./08-factories.md) · [Topic](./README.md) · [Next →](./10-seeder-vs-factory.md)
