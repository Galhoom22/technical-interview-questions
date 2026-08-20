# What are the general types of APIs?

> Laravel examples target **Laravel 13**. Official docs: [https://laravel.com/framework/docs](https://laravel.com/framework/docs)
>
> PHP examples use **PHP 8.5** syntax. Official manual: [https://www.php.net/manual/en/](https://www.php.net/manual/en/)

**Answer:**

“API” just means a contract for programs to talk. The types I name in interviews:

| Type | Style | Typical transport |
|------|--------|-------------------|
| **REST** | Resources + HTTP verbs + status codes | HTTP + JSON |
| **RPC** | Call a procedure (`createOrder`) | HTTP, gRPC |
| **GraphQL** | Client asks for the exact fields | HTTP + one `/graphql` endpoint |
| **SOAP** | XML envelopes, WSDL contracts | HTTP + XML |
| **WebSocket / SSE** | Push / streaming, not request-response only | WS / HTTP |

Also useful split: **public HTTP APIs** vs **library APIs** (a PHP package’s classes). Interviewers almost always mean HTTP.

I default to REST for CRUD backends. GraphQL when many clients need different shapes. gRPC for internal service-to-service with strict contracts. SOAP when an enterprise partner still requires it.

> [!TIP]
> **One-liner:** Common API styles are REST, RPC/gRPC, GraphQL, SOAP, and realtime (WebSocket). REST over HTTP + JSON is the default for Laravel backends.

**Source:** [MDN: REST](https://developer.mozilla.org/en-US/docs/Glossary/REST) — what “REST” actually means vs “any HTTP JSON API”.

**Learn more:**
- [Fielding: REST architectural style](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) — the original dissertation chapter
- [GraphQL: Learn](https://graphql.org/learn/) — when a single `/graphql` endpoint is a better fit
- [gRPC: Core concepts](https://grpc.io/docs/what-is-grpc/core-concepts/) — RPC over HTTP/2 for service-to-service

---

[← Previous](./02-status-codes.md) · [Topic](./README.md) · [Next →](./04-restful-api.md)
