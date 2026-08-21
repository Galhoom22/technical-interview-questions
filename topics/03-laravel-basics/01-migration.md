# What is a Migration?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

A migration is a **version-controlled PHP class that changes the database schema**. It is Git for tables: `up()` applies the change, `down()` reverses it.

**Official** ([Laravel: Migrations](https://laravel.com/docs/13.x/migrations)):

```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

```bash
php artisan make:migration create_flights_table
php artisan migrate
php artisan migrate:rollback
```

**In production:**

```php
public function up(): void
{
    Schema::create('orders', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->restrictOnDelete();
        $table->unsignedInteger('total_cents');
        $table->char('currency', 3);
        $table->string('status');
        $table->timestamps();
    });
}
```

Laravel records ran migrations in a `migrations` table, so each change runs once per environment.

**Why it matters:** the team shares schema through Git instead of manually clicking phpMyAdmin. Staging and production stay in sync with the same files.

> [!TIP]
> **One-liner:** A migration is a versioned schema change (`up` / `down`) so every environment applies the same database structure from Git.

**Source:** [Laravel: Migrations](https://laravel.com/docs/13.x/migrations) — official `up` / `down`, `migrate`, rollback, and the `migrations` table.

**Learn more:**
- [Laravel: Database](https://laravel.com/docs/13.x/database) — connections, transactions, query logging
- [Laravel: Eloquent](https://laravel.com/docs/13.x/eloquent) — models that sit on top of the tables you migrate
- [PostgreSQL: Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html) — UNIQUE, FK, CHECK (what the migration is encoding)

---

[Topic](./README.md) · [Next →](./02-routing.md)
