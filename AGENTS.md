# Agent notes

This is an **interview answer bank**, not a course and not a scraped question dump. Every item is a question the owner was actually asked. Answers are English. Examples target **PHP 8.5** and **Laravel 13**.

Root is `README.md` (humans) + `topics/` + this file. Do not add LICENSE, CONTRIBUTING, homework, study cards, PLAN/phases, a full Laravel app, or extra questions to “fill” a topic.

## Do not

- Invent topics or questions so the table looks balanced. Uneven counts are correct.
- Reorder folders or files for chronology or aesthetics. Numbers are **study order**.
- Put emoji in question titles (`#`). Topic index titles may keep their emoji.
- Promote `**Answer:**` to `## Answer`. It stays body text under the H1.
- Replace `###` section headings with bold labels.
- Translate question files into Arabic.

## Layout

```text
topics/NN-kebab-topic/
  README.md          # index + prev/next topic
  NN-kebab-slug.md   # one question
```

Study order of topics: PHP → OOP → HTTP/REST → Laravel → Database → Security → Testing → Git → Refactoring → Performance.

Same-topic files are also study order. Keep Previous / Next / Topic footers correct if you rename.

## Question file shape

1. `#` question (exact interview wording)
2. Version banner (PHP 8.5 and/or Laravel 13) with official docs link
3. `**Answer:**` short definition (tables OK)
4. `### 🔑 Key terms` — Term | Plain meaning, 3–6 rows; reuse wording across sibling files
5. `### 🧠 Analogy` — 3–5 lines
6. `---` then `### 📘 Official ([Name](url))` — snippet from the cited doc, not paraphrased trivia
7. `### 💼 In production` — e-commerce/backend (orders, cart, checkout, Sanctum, Form Requests)
8. `---` then `> [!WARNING]` — the interview trap (no extra `###` heading; the alert is the label)
9. `> [!NOTE]` — follow-up questions, 1–2 bullets (same: no extra `###`)
10. `---` then `> [!TIP]` one-liner, `**Source:**`, `**Learn more:**`, `---`, nav

Fences always have a language tag (`php`, `sql`, `html`, `javascript`, `text`, …).

Exactly **three** `---` phase breaks in the body (after analogy, after production, before TIP). Do not put `---` after every section. Do not double the nav rule.

Official snippets: PHP Manual, Laravel 13 docs, MDN, RFC 9110, OpenAPI, git-scm, GitHub Docs, Fowler, PostgreSQL. Match the cited page. Production code stays invented-but-realistic, not copied from the manual.

## Topic `README.md`

Keep the one-line legend + question list + one prev/next topic line. Link the legend to `../../README.md`.

## New question

Only if the owner says they were asked it. Append in study order, number the file, update the topic index and nav, copy the shape above. Do not add a topic that has no real question.
