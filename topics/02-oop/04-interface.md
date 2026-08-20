# What is an Interface, and why is it used?

> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

An interface is a **contract**: a list of public methods a class **must** implement. It has no state (historically no properties) and does not care *how* the work is done.

```php
interface Mailer
{
    public function send(string $to, string $subject, string $body): void;
}

class SmtpMailer implements Mailer
{
    public function send(string $to, string $subject, string $body): void
    {
        // SMTP implementation
    }
}

class LogMailer implements Mailer
{
    public function send(string $to, string $subject, string $body): void
    {
        logger()->info("Mail to {$to}: {$subject}");
    }
}
```

**Why we use it:**

- **Decoupling** — controllers depend on `Mailer`, not `SmtpMailer`.
- **Polymorphism** — any implementation can be swapped (real SMTP in prod, fake in tests).
- **Multiple contracts** — a class can `implement` many interfaces; it can `extend` only one class.
- **Clear API** — the interface is the public surface other code is allowed to rely on.

In Laravel this is the Service Container pattern: bind `Mailer::class` to `SmtpMailer::class`.

> [!TIP]
> **One-liner:** An interface is a contract of public methods. You use it so code depends on *behavior*, not a specific class — which makes swapping and testing easy.

**Source:** [PHP: Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php) — official contract, `implements`, and multiple interfaces.

**Learn more:**
- [Laravel: Service Container](https://laravel.com/docs/13.x/container) — binding an interface to a class (the pattern used in real Laravel apps)
- [Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php) — when a partial implementation is a better fit than a pure interface
- [PHP 8.5 new features](https://www.php.net/manual/en/migration85.new-features.php) — current syntax (`|>`, `clone()`, `#[\Override]` on properties). Property hooks on interfaces shipped in 8.4 and remain in 8.5.

---

[← Previous](./03-what-is-a-class.md) · [Topic](./README.md) · [Next →](./05-trait.md)
