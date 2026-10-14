# Technical Plan — Manager Library (Laravel)

**Status: Ready for implementation.** This plan contains the approved Laravel-target choices. It does not change product behavior in [`spec.md`](spec.md).

## 1. Technical decisions

| ID | Area | Decision | Rationale | Status |
|---|---|---|---|---|
| D-01 | Backend framework and runtime | Laravel 13 on PHP 8.3 or later | Pins the target to the user-approved Laravel release and its minimum PHP version ([official deployment requirements](https://laravel.com/docs/13.x/deployment)) | Approved |
| D-02 | Persistence and migrations | PostgreSQL 16; use schema `library`; Laravel migrations own all DDL and the fixed category seed | One versioned schema source; infrastructure starts with an empty database; see [ADR-001](adr/adr-001-schema-ownership.md) | Approved |
| D-03 | Concurrency control | In one loan transaction, lock the member row first, then requested book rows in ascending ID order; no version field | Serializes the member's aggregate-loan limit and prevents concurrent loans from overselling copies; see [ADR-002](adr/adr-002-concurrency-control.md) | Approved |
| D-04 | Authentication and password hashing | Signed JWT using HS256 for 8 hours; store passwords as bcrypt hashes with factor 12 | Matches the API contract and the approved .NET reference choice for password hashing | Approved |
| D-05 | Cover-file storage | Local disk in development; S3 in production; store only the opaque key in PostgreSQL | Keeps local setup simple and production covers durable and externally stored; uses Laravel's filesystem disk abstraction ([official filesystem documentation](https://laravel.com/docs/13.x/filesystem)) | Approved |
| D-06 | Testing strategy | PHPUnit, run through `php artisan test`; use PostgreSQL for database-specific integration and concurrency tests | Uses Laravel's built-in PHPUnit support and verifies behavior that isolated fakes cannot prove ([official testing documentation](https://laravel.com/docs/13.x/testing)) | Approved |

## 2. Required configuration

Secrets and environment-specific values must be provided through the deployment environment and must not be committed. Exact variable names should be aligned with the API and infrastructure repositories before implementation.

| Setting | Requirement |
|---|---|
| Database connection | PostgreSQL 16 connection; the database name is supplied by infrastructure and the application schema is `library` |
| JWT signing secret | Required; sufficiently random and never logged or committed |
| Initial administrator | Username and password supplied at deployment; password hashed before persistence |
| Frontend origin | Explicit origin used by the CORS allowlist |
| Cover storage | Use the local disk for development and the configured S3 disk in production; provide S3 credentials through the deployment environment |

## 3. Verification strategy

The PHPUnit suite, run with `php artisan test`, must demonstrate:

| Level | Required evidence |
|---|---|
| Domain and application | Loan limits, due dates, status calculation, member restrictions, and report grouping |
| HTTP/API | Authentication, role authorization, request validation, response contracts, and error envelopes |
| Persistence integration | PostgreSQL migrations, foreign keys, unique and check constraints, partial ISBN uniqueness, and report query |
| Concurrency integration | Two simultaneous requests for the last copy yield one 201 and one 409, with no partial loan |
| Aggregate concurrency integration | Concurrent loans for a member near the three-item limit never exceed that limit; concurrent overdue checks do not bypass the overdue rule |
| File handling | JPEG/PNG and size validation, opaque stored key, and upload/retrieval/deletion response contract |
| Architecture review | Domain and application do not import HTTP, Eloquent, filesystem, or JWT adapter implementations |

Use PostgreSQL for tests that exercise row locks, partial indexes, or other PostgreSQL-specific behavior. Isolated fakes may be used for application tests that do not claim to prove database behavior.

## 4. Build sequence

After the documentation readiness gate is approved, the implementation sequence is: Laravel application skeleton and health endpoint → schema migrations and fixed categories → domain and application use cases → persistence, security, and storage adapters → API routes and contract behavior → integration and concurrency verification. The user-story task breakdown and completion evidence live in [`tasks.md`](tasks.md).
