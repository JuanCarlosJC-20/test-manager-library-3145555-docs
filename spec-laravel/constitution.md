# Project Constitution — Manager Library (Laravel)

**Status: Ready for implementation.** These principles govern the Laravel specification and must remain consistent with the product requirements and accepted architecture decisions.

## 1. Product scope and sources of truth

- `spec.md` owns product behavior and acceptance criteria. Do not change the product scope when adapting the technical references to Laravel.
- `data-model.md` owns persistent entities, fields, constraints, and indexes.
- `api-contract.md` owns HTTP routes, request and response fields, authorization, and error formats.
- `architecture.md` owns component boundaries and dependency direction.
- `plan.md` owns target-stack choices and verification strategy.
- Accepted ADRs record decisions that affect architecture or data behavior. When a decision changes, update its ADR and all affected documents together.

## 2. Language and naming

- Explanatory prose, headings, tables, and technical descriptions are written in English.
- Spanish category labels are product data and remain unchanged. User-facing error messages are Spanish as required by RN-6; code identifiers and documentation explanations remain English.
- Database tables use English plural `snake_case` names with no entity prefix, for example `books` and `loan_items`.
- Database columns, foreign-key columns, and JSON fields use English `snake_case`, for example `member_id` and `total_copies`.
- PHP classes and Eloquent models use singular `PascalCase`, for example `Book` and `LoanItem`. Methods and local variables use `camelCase`.
- HTTP collection routes use English plural nouns, for example `/books` and `/members`.

| Area | Convention | Example |
|---|---|---|
| Database table | English plural `snake_case`; no prefix | `loan_items` |
| Database column | English `snake_case` | `due_at` |
| JSON field | English `snake_case` | `available_copies` |
| PHP class / model | Singular `PascalCase` | `LoanItem` |
| PHP method / variable | `camelCase` | `returnItem` |
| HTTP collection route | English plural | `/loan-items` |
| Entity prefix | None | `books`, not `lib_books` |

## 3. Architecture principles

- Use hexagonal architecture adapted to Laravel: the domain and application core do not depend on HTTP, persistence, file-storage, or token infrastructure.
- Keep Laravel-specific types at the relevant adapters. Do not copy .NET project names, types, or folder paths literally.
- The application defines ports for external capabilities it consumes; adapters implement those ports. Introduce abstractions where they protect a real boundary, rather than creating empty interfaces for every class.
- Keep dependency wiring in Laravel's composition mechanisms and make the bindings explicit and discoverable.
- Enforce product invariants in the application/domain and use database constraints as the final integrity guard where the database can express them.
- Create a loan and decrement available copies atomically, using the approved database row-lock strategy in ADR-002.

## 4. Schema, data, and security

- Laravel database migrations are the sole versioned source of truth for schema changes. The initial migration also inserts the fixed category reference data as approved in ADR-001.
- Store UTC instants and use UUID v4 identifiers as specified in `data-model.md`.
- Store passwords only as bcrypt hashes with factor 12. Never return, log, or otherwise expose a password or its hash.
- Keep cover binaries in configured storage and store only the opaque key in the database.
- Keep Spanish category labels as product data, while all explanatory documentation remains in English.

## 5. Readiness and change control

- Keep the specification implementation-ready only while its critical choices are resolved and its documents agree.
- Do not begin application implementation until the user stories, data model, API contract, architecture, technical plan, tasks, and ADRs agree.
- Any approved decision that changes behavior or system boundaries must be recorded in an ADR and reconciled across the affected documents before implementation.
