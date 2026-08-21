# How would you approach designing a database for an e-commerce platform? Describe your initial steps for mapping out the entities and relationships.

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)

**Answer:**

I start from **what the business does**, not from tables. Tables come after entities and relationships are clear.

### 🔑 Key terms

| Term | Plain meaning |
|------|----------------|
| Entity | A noun in the domain that becomes a table (`Order`, `Product`). |
| Foreign key | A column that must match a row in another table. |
| Snapshot | Copy price/name onto `order_items` so the invoice does not change when the catalog does. |
| Cardinality | How many: `1──*` (one-to-many), `*──*` (many-to-many). |
| 3NF | No repeating groups; facts live in the table they belong to. |

### 🧠 Analogy

I draw the **shop story** first (browse, cart, pay), then the nouns, then the arrows. Tables are last. `order_items` takes a **snapshot** of price so yesterday’s invoice does not change when you discount the T-shirt tomorrow.

**Step 1 — Use cases, not columns**

- Guest and user browse catalog
- Add to cart, change qty, apply coupon
- Checkout: address, shipping, payment
- Order history, refunds, stock, admin catalog

If a use case is missing, the schema will be missing too.

**Step 2 — Entities (nouns)**

A first pass I would put on a whiteboard:

| Entity | Role |
|--------|------|
| `users` | Accounts |
| `addresses` | Shipping / billing |
| `categories` / `products` | Catalog |
| `product_variants` | Size/color, own SKU and price |
| `inventory` | Stock per variant (or warehouse) |
| `carts` / `cart_items` | Current basket (guest or user) |
| `orders` / `order_items` | Placed purchase (**snapshot** of price/name) |
| `payments` | Charge attempts, status, provider id |
| `coupons` | Optional discounts |

**Step 3 — Relationships**

---

### 📘 Official ([PostgreSQL: Foreign Keys](https://www.postgresql.org/docs/current/tutorial-fk.html))

```sql
CREATE TABLE cities (
    name     varchar(80) primary key,
    location point
);

CREATE TABLE weather (
    city      varchar(80) references cities(name),
    temp_lo   int,
    temp_hi   int,
    date      date
);
```

A row in `weather` cannot name a city that does not exist. That is referential integrity — the same idea as `orders.user_id` → `users.id`.

### 💼 In production

```text
User 1──* Address
User 1──* Order
User 1──1 Cart

Product 1──* Variant 1──* Inventory
Cart 1──* CartItem *──1 Variant
Order 1──* OrderItem *──1 Variant   (item stores price snapshot)
Order 1──* Payment
```

Cardinality I say out loud: many products per category (or many-to-many if a product has several categories — then a pivot `category_product`).

**Step 4 — Rules that change the schema**

- **Orders do not join live prices.** `order_items` copies name, SKU, unit price. Catalog can change tomorrow; the invoice cannot.
- **Guest cart** is keyed by `session_id` (or cookie) until login, then merged (see [Security Q2](../06-security/02-guest-cart-merge-on-login.md)).
- **Stock** is decremented on checkout in a transaction, not “when they add to cart” unless the business wants reservations.
- Money as **integer cents** (or `decimal(12,2)` with a documented currency). Never float.

**Step 5 — Then normalize, then indexes**

- 3NF for operational tables: no repeating groups, FKs for relations.
- Indexes on FKs and hot filters: `orders.user_id`, `products.slug`, `order_items.order_id`.
- Unique: `users.email`, `variants.sku`, `payments.provider_reference`.

I would sketch this as an ERD (even on paper) and only then write Laravel migrations. I do not start with 40 tables; I start with catalog + cart + order, and add warehouse/refunds when the use case is real.

---

> [!WARNING]
> Never store money as float. Never join live `products.price` for an old order. Do not start from 40 tables.

> [!NOTE]
> - Guest cart? `session_id` until login, then merge ([Security Q2](../06-security/02-guest-cart-merge-on-login.md)).
> - Stock? Decrement in a checkout transaction, not “on add to cart” unless the business wants reservations.

---

> [!TIP]
> **One-liner:** List use cases, name entities, draw cardinality, snapshot prices on `order_items`, then normalize and add FKs/indexes — migrations last, not first.

**Source:** [PostgreSQL: Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html) — UNIQUE, foreign keys, CHECK (how the ERD becomes a schema). 1NF/2NF/3NF terminology: [Database normalization](https://en.wikipedia.org/wiki/Database_normalization) (standard teaching reference, not an ISO spec).

**Learn more:**
- [Laravel: Eloquent Relationships](https://laravel.com/docs/13.x/eloquent-relationships) — how those entities become `hasMany` / `belongsToMany`
- [Laravel: Migrations](https://laravel.com/docs/13.x/migrations) — FKs, unique indexes, `morphs()`
- [PostgreSQL: Foreign Keys](https://www.postgresql.org/docs/current/tutorial-fk.html) — referential integrity in practice
- [PostgreSQL: Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html) — prove indexes after the schema exists

---

[Topic](./README.md)
