# Laravel / PHP Backend Developer Interview Questions

Real **technical** questions from real interviews I sat, with the answers I would give, plus official sources to go deeper.

This is not a scraped question bank. Every item here is one I was actually asked. There is no HR / behavioral content.

**46 questions** across 10 topics. Examples use **PHP 8.5** and **Laravel 13**. Last reviewed: August 2026.

Sources in this repo are official or primary references: [PHP Manual](https://www.php.net/manual/en/), [Laravel 13 docs](https://laravel.com/framework/docs), [MDN](https://developer.mozilla.org/), [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html), [PHP-FIG](https://www.php-fig.org/), [OWASP](https://owasp.org/), [GitHub Docs](https://docs.github.com/), [Git Book](https://git-scm.com/book/en/v2).

| Stack | Version | Official docs |
|-------|---------|---------------|
| PHP | 8.5 | [PHP Manual](https://www.php.net/manual/en/) |
| Laravel | 13 | [Laravel Documentation](https://laravel.com/framework/docs) |

[PHP 8.5 release notes](https://www.php.net/releases/8.5/en.php) · [PHP 8.5 new features](https://www.php.net/manual/en/migration85.new-features.php) · [Laravel 13.x docs](https://laravel.com/docs/13.x)

## Topics

| # | Topic | Questions |
|---|-------|-----------|
| 01 | [PHP - Basics](./01-php-basics.md) | 4 |
| 02 | [OOP](./02-oop.md) | 16 |
| 03 | [Laravel - Basics](./03-laravel-basics.md) | 10 |
| 04 | [HTTP & REST API](./04-http-rest-api.md) | 9 |
| 05 | [Git & Workflow](./05-git-workflow.md) | 1 |
| 06 | [Testing](./06-testing.md) | 1 |
| 07 | [Refactoring & Best Practices](./07-refactoring-best-practices.md) | 1 |
| 08 | [Database Design](./08-database-design.md) | 1 |
| 09 | [Performance & Optimization](./09-performance-optimization.md) | 1 |
| 10 | [Security](./10-security.md) | 2 |

## How to use this

1. Pick a topic.
2. Read the question and answer it out loud for about a minute.
3. Compare with the model answer and the **one-liner** at the bottom.
4. Open **Source**, then **Learn more**, if you want the follow-up material interviewers often ask next.

Each answer is written to be said in an interview: short definition first, then a table or example, then practical points. **Source** and **Learn more** are there so one question also teaches the next layer (the spec, the Laravel 13 default, the follow-up they ask).

## What you will find in an answer

- A direct opening (the sentence that answers the question)
- A comparison table or example when it helps
- PHP 8.5 / Laravel 13 code where it matters
- An interview **one-liner**
- **Source** — the official or primary page the answer is based on
- **Learn more** — extra official docs (community pages are labeled as such)

Version claims are pinned: `match` is PHP 8.0+, `array_is_list()` is PHP 8.1+, `array_first()` / `array_last()` / `|>` / `clone($obj, [...])` are PHP 8.5. Laravel 13 API routes use `php artisan install:api` (Sanctum), not Passport, as the default.

## Maintainer

[Abdelrahman Galhoom](https://github.com/Galhoom22) — backend developer (Laravel / PHP).
