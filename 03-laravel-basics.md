# 🚀 Laravel - Basics

## Q1. What is a Migration?

**Answer:**

A migration is a **version-controlled PHP class that changes the database schema**. It is Git for tables: `up()` applies the change, `down()` reverses it.

```php
public function up(): void
{
    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('email')->unique();
        $table->timestamps();
    });
}

public function down(): void
{
    Schema::dropIfExists('users');
}
```

```bash
php artisan make:migration create_users_table
php artisan migrate
php artisan migrate:rollback
```

Laravel records ran migrations in a `migrations` table, so each change runs once per environment.

**Why it matters:** the team shares schema through Git instead of manually clicking phpMyAdmin. Staging and production stay in sync with the same files.

> [!TIP]
> **One-liner:** A migration is a versioned schema change (`up` / `down`) so every environment applies the same database structure from Git.

---

## Q2. What is Routing?

**Answer:**

Routing maps an **HTTP request** (method + URI) to the code that should handle it: a closure, a controller action, or an invokable controller.

```php
Route::get('/users/{user}', [UserController::class, 'show']);
Route::post('/users', [UserController::class, 'store']);
```

The router:

1. Matches method + path (and optional name, middleware, where constraints).
2. Runs route middleware.
3. Resolves the controller from the container (dependency injection).
4. Performs route-model binding (`{user}` → `User` model).
5. Returns the response.

Named routes (`->name('users.show')`) keep `route()` URLs from going stale when the path changes.

> [!TIP]
> **One-liner:** Routing is the map from “HTTP method + URL” to the controller (or closure) that handles that request.

---

## Q3. What is the `web.php` file?

**Answer:**

`routes/web.php` is where **browser / session** routes live. In a default Laravel app they are loaded in the `web` middleware group.

That group typically gives you:

| Feature | Why it is on web routes |
|---------|-------------------------|
| Cookies + session | Logged-in users, flash data |
| CSRF protection | HTML forms must send a token |
| Cookie encryption | Session cookie is not plaintext |
| `SubstituteBindings` | Route-model binding |

```php
// routes/web.php
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('auth')
    ->name('dashboard');
```

These routes are for Blade / Inertia / Livewire pages, not for a stateless mobile API.

> [!TIP]
> **One-liner:** `web.php` holds session-based browser routes — cookies, CSRF, and auth via the `web` middleware group.

---

## Q4. What is the `api.php` file?

**Answer:**

`routes/api.php` holds **stateless API** routes. They are usually prefixed with `/api` and loaded in the `api` middleware group (throttling, JSON, no session CSRF).

```php
// routes/api.php
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

| | `web.php` | `api.php` |
|---|-----------|-----------|
| Client | Browser | App / SPA / third party |
| Auth | Session cookie | Token (Sanctum / Passport) |
| CSRF | Yes | No (stateless) |
| URL | `/dashboard` | `/api/user` |
| Typical response | HTML | JSON |

**Laravel 11+:** `api.php` is not always there on install. You add it with `php artisan install:api`, which also publishes the API route file and Sanctum if you choose it.

> [!TIP]
> **One-liner:** `api.php` is for stateless JSON APIs — `/api` prefix, rate limiting, token auth, no session CSRF.

---

## Q5. Explain the different types of Eloquent relationships in Laravel.

**Answer:**

A relationship is a method on the model that tells Eloquent how two tables connect. It returns a relation object; you query through it or eager-load it with `with()`.

| Relationship | DB shape | Example |
|--------------|----------|---------|
| `hasOne` | Other table holds this id (one row) | User `hasOne` Profile |
| `hasMany` | Other table holds this id (many rows) | User `hasMany` Posts |
| `belongsTo` | This table holds the other id | Post `belongsTo` User |
| `belongsToMany` | Pivot table | Post `belongsToMany` Tag |
| `hasManyThrough` | Related through an intermediate | Country `hasManyThrough` Post via User |
| `hasOneThrough` | Same idea, one row | Supplier `hasOneThrough` History via User |
| `morphOne` / `morphMany` | Polymorphic one/many | Comment `morphTo`; Post/Video `morphMany` Comments |
| `morphTo` | Inverse of morph | Comment belongs to Post **or** Video |
| `morphToMany` / `morphedByMany` | Polymorphic many-to-many | Tag attached to Post and Video |

```php
class User extends Model
{
    public function profile(): HasOne
    {
        return $this->hasOne(Profile::class);
    }

    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }

    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}

class Comment extends Model
{
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

**Practical points for the interview:**

- Always define **both sides** (`hasMany` + `belongsTo`) when both are queried.
- Avoid N+1: `User::with('posts.tags')->get()`.
- Pivot extras: `->withPivot('approved')->withTimestamps()`.
- Foreign key names default to `{model}_id`. Override when the column is not conventional.

> [!TIP]
> **One-liner:** Eloquent relationships map table links: `hasOne` / `hasMany` / `belongsTo`, `belongsToMany` (pivot), `hasManyThrough`, and `morph*` for polymorphic relations.

---

## Q6. What is Middleware, and what are its primary use cases?

**Answer:**

Middleware is a **filter around the HTTP request**. It can inspect, reject, or enrich the request before the controller, and it can tweak the response on the way out.

```
request → middleware → middleware → controller → middleware → response
```

```php
class EnsureUserIsAdmin
{
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->isAdmin()) {
            abort(403);
        }

        return $next($request);
    }
}
```

**Primary use cases:**

| Use case | Example |
|----------|---------|
| Authentication | `auth`, `auth:sanctum` |
| Authorization | `can:update,post` |
| CSRF | `ValidateCsrfToken` on web |
| Rate limiting | `throttle:api` |
| Localization | set locale from header |
| Logging / CORS | log request, add CORS headers |
| Maintenance | `PreventRequestsDuringMaintenance` |

Registration:

- **Global** — every request
- **Group** — `web` vs `api`
- **Route** — one route or a `Route::middleware([...])` group

Laravel 11+ wires this in `bootstrap/app.php` instead of `Http/Kernel.php`.

> [!TIP]
> **One-liner:** Middleware is request/response middleware — auth, CSRF, throttle, CORS — that runs before (and after) the controller.

---

## Q7. What is the difference between Eloquent ORM and the Query Builder?

**Answer:**

**Query Builder** (`DB::table(...)`) builds SQL and returns plain data (arrays / `stdClass`). **Eloquent** is an ORM on top of that builder: each row is a **model** with relationships, casts, events, and mutators.

| Aspect | Query Builder | Eloquent |
|--------|---------------|----------|
| Entry | `DB::table('users')` | `User::query()` |
| Result | Arrays / objects, not models | `User` models / collections |
| Relationships | You join by hand | `with()`, relation methods |
| Casts / hidden / appends | No | Yes |
| Model events / observers | No | Yes |
| Overhead | Lower | Higher (hydration) |
| Best for | Reports, aggregates, bulk SQL | Domain entities, CRUD |

```php
// Query Builder
$emails = DB::table('users')->where('active', 1)->pluck('email');

// Eloquent
$users = User::with('posts')->where('active', true)->get();
```

Eloquent **uses** the Query Builder internally (`User::query()` is a builder). Choosing Eloquent vs `DB::` is about whether you need a model or just SQL.

I use Eloquent for domain work. I drop to Query Builder (or `selectRaw`) for heavy reports, bulk updates, and queries where hydrating thousands of models would be wasteful.

> [!TIP]
> **One-liner:** Query Builder is SQL in PHP. Eloquent is that builder plus models — relationships, casts, and events. Eloquent sits on top of the Query Builder.

---

## Q8. What are Factories?

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
            'password' => 'hashed-password',
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

---

## Q9. What is a Seeder?

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

---

## Q10. What is the difference between a Seeder and a Factory?

**Answer:**

They work together. They are not the same layer.

| | Factory | Seeder |
|---|----------|--------|
| Role | **How one model looks** when fake | **What gets inserted**, and how many |
| Runs | From tests or from a seeder | `php artisan db:seed` |
| Knows about | One model (and related factories) | Whole database / feature set |
| Example | `UserFactory` definition + `admin()` state | “Create 3 roles, 1 admin, 50 users” |

```php
// Factory: the recipe
User::factory()->admin()->create();

// Seeder: the meal plan
public function run(): void
{
    Role::factory()->create(['name' => 'admin']);
    User::factory()->admin()->create(['email' => 'admin@example.com']);
    User::factory()->count(50)->create();
}
```

Tests usually call **factories directly** (fast, isolated). Seeders are for **environments** (local demo, staging). Production rarely runs dummy factories; it may run a seeder for required lookup rows only.

> [!TIP]
> **One-liner:** A factory describes one fake model. A seeder decides which records to insert. Seeders often call factories.
