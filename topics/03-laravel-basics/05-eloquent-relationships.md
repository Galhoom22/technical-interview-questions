# Explain the different types of Eloquent relationships in Laravel.

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

A relationship is a method on the model that tells Eloquent how two tables connect. It returns a relation object; you query through it or eager-load it with `with()`.

**🔑 Key terms:**

| Term | Plain meaning |
|------|----------------|
| Relationship | A method that tells Eloquent how two tables connect. |
| Eager loading | `with()` so related rows load in fewer queries (avoid N+1). |
| N+1 | One query plus one query per row because a relation was lazy-loaded. |
| Pivot | Extra table for `belongsToMany`. |
| Polymorphic | One relation can point at more than one model type (`morphTo`). |

**🧠 Analogy:**

A relationship is a **labeled arrow** between tables: User → many Orders. `with()` loads the arrows in a few queries instead of one query per row (N+1).

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

**📘 Official** ([Laravel: Eloquent Relationships](https://laravel.com/docs/13.x/eloquent-relationships)):

```php
class User extends Model
{
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}

class Post extends Model
{
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

**💼 In production:**

```php
class User extends Model
{
    public function cart(): HasOne
    {
        return $this->hasOne(Cart::class);
    }

    public function orders(): HasMany
    {
        return $this->hasMany(Order::class);
    }
}

class Order extends Model
{
    public function items(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }

    public function products(): BelongsToMany
    {
        return $this->belongsToMany(Product::class, 'order_items')
            ->withPivot('qty', 'unit_cents');
    }
}

User::with('orders.items')->paginate(20); // avoid N+1 on the order history page
```

**Practical points for the interview:**

- Always define **both sides** (`hasMany` + `belongsTo`) when both are queried.
- Avoid N+1: `User::with('posts.tags')->get()`. In local/staging, `Model::preventLazyLoading(! app()->isProduction())` makes missed eager loads throw instead of silently multiplying queries.
- Pivot extras: `->withPivot('approved')->withTimestamps()`.
- Foreign key names default to `{model}_id`. Override when the column is not conventional.

**⚠️ Watch out:**

N+1 is the follow-up they always run. Define **both sides** (`hasMany` + `belongsTo`) when both are queried. Wrong FK name silently returns empty.

**💬 If they follow up:**

- How do you catch N+1 locally? `Model::preventLazyLoading(! app()->isProduction())`.
- Pivot extras? `withPivot` / `withTimestamps`.

> [!TIP]
> **One-liner:** Eloquent relationships map table links: `hasOne` / `hasMany` / `belongsTo`, `belongsToMany` (pivot), `hasManyThrough`, and `morph*` for polymorphic relations.

**Source:** [Laravel: Eloquent Relationships](https://laravel.com/docs/13.x/eloquent-relationships) — official catalog of every relation type in this answer.

**Learn more:**
- [Eager Loading](https://laravel.com/docs/13.x/eloquent-relationships#eager-loading) — `with()` / N+1 (the follow-up they always ask)
- [Preventing Lazy Loading](https://laravel.com/docs/13.x/eloquent-relationships#preventing-lazy-loading) — `Model::preventLazyLoading()` (Laravel 13 current practice)
- [Laravel: Migrations](https://laravel.com/docs/13.x/migrations) — `foreignId()`, `morphs()`, pivot tables
- [Polymorphic relationships](https://laravel.com/docs/13.x/eloquent-relationships#polymorphic-relationships) — `morphTo` / `morphMany` in isolation

---

[← Previous](./04-api-php.md) · [Topic](./README.md) · [Next →](./06-middleware.md)
