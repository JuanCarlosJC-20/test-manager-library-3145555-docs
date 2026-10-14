# HTTP API Contract — Manager Library

**Status: Draft.** This is the single source of truth for routes, request and response fields, authorization, and HTTP errors. Product behavior comes from [`spec.md`](spec.md); persistent fields come from [`data-model.md`](data-model.md).

## 1. Conventions

| Topic | Rule |
|---|---|
| Versioned API base | `/api/v1`; endpoint paths below are relative to this base unless identified as unversioned |
| Health endpoint | Unversioned `GET /health` |
| Format | JSON encoded as UTF-8; field names use `snake_case` |
| Identifiers | UUID v4 strings |
| Timestamps | ISO 8601 UTC instants with `Z`, for example `2026-10-14T14:15:00Z` |
| Pagination | `?page=1&size=20`; maximum `size` is 50. Responses use `{ "items": [...], "page": 1, "size": 20, "total": 134 }` |
| Error language | User-facing error messages and validation issue text are in Spanish; field names and error codes are in English |

### Authentication

Send `Authorization: Bearer <token>` on every endpoint except `POST /auth/login` and unversioned `GET /health`.

### Error response body

Every HTTP error uses this shape:

```json
{
  "code": "BOOK_NOT_AVAILABLE",
  "message": "No quedan ejemplares disponibles de «Cien años de soledad».",
  "details": [
    { "field": "items[0].book_id", "issue": "sin ejemplares" }
  ]
}
```

`details` may be an empty array. Responses never include stack traces, SQL, server paths, passwords, or password hashes.

### HTTP status codes

| Status | Meaning |
|---|---|
| 200 | Successful read or operation with a response body |
| 201 | Resource created |
| 204 | Successful operation with no response body |
| 401 | Missing, invalid, or expired token; or invalid credentials |
| 403 | The authenticated role is insufficient |
| 404 | Resource does not exist or is soft-deleted |
| 409 | State conflict: duplicate, no available copy, item already returned, member has overdue items, or book has open loans |
| 422 | Invalid input or business validation failure, including an invalid date range or file upload |

## 2. Business error codes

| Code | Status | Condition |
|---|---:|---|
| `INVALID_CREDENTIALS` | 401 | Invalid sign-in credentials (CA-01.2) |
| `FORBIDDEN_ROLE` | 403 | Authenticated role cannot perform the operation (CA-01.3) |
| `NOT_FOUND` | 404 | Requested resource does not exist or is soft-deleted |
| `USERNAME_TAKEN` | 409 | Username already exists (CA-02.2) |
| `ISBN_TAKEN` | 409 | ISBN already belongs to an active book (CA-03.2) |
| `DOCUMENT_TAKEN` | 409 | Member document number already exists (CA-05.1) |
| `BOOK_HAS_OPEN_LOANS` | 409 | Book cannot be soft-deleted while it has open loan items (CA-03.5) |
| `BOOK_NOT_AVAILABLE` | 409 | Requested book has no available copy (CA-06.3) |
| `MEMBER_HAS_OVERDUE` | 409 | Member has at least one overdue item (CA-06.5) |
| `ITEM_ALREADY_RETURNED` | 409 | Loan item was already returned (CA-07.2) |
| `MEMBER_LIMIT_EXCEEDED` | 422 | Loan would exceed the member's unreturned-book limit (CA-06.4) |
| `DUPLICATE_BOOK_IN_LOAN` | 422 | Same book appears more than once in one loan (CA-06.2) |
| `VALIDATION_ERROR` | 422 | Other invalid field or request data |
| `INVALID_DATE_RANGE` | 422 | Report end date precedes start date (CA-10.2) |

## 3. Endpoints

Successful responses follow the approved REST convention: creation returns `201` with the created resource, resource updates return `200` with the updated resource, and deletions return `204` without a body. Cover upload and deletion return `204`; cover retrieval returns `200` with image bytes.

### 3.1 Authentication and health

| Method | Path | Access | Success |
|---|---|---|---|
| POST | `/auth/login` | Public | 200; returns a token |
| GET | `/health` (unversioned) | Public | 200; `{ "status": "ok" }` |

`POST /api/v1/auth/login` request and response:

```json
{ "username": "ana", "password": "********" }
```

```json
{ "access_token": "eyJ...", "token_type": "Bearer", "expires_in": 28800, "role": "librarian" }
```

### 3.2 Users

| Method | Path | Access | Purpose | Success |
|---|---|---|---|---|
| POST | `/users` | `admin` | Create a librarian account | 201; created user representation |

Request:

```json
{ "username": "Ana ", "password": "********", "role": "librarian" }
```

The API accepts only `librarian` as a requested role. The response must not contain `password` or `password_hash`.

Representative response:

```json
{ "id": "…", "username": "ana", "role": "librarian" }
```

### 3.3 Categories

| Method | Path | Access | Purpose | Success |
|---|---|---|---|---|
| GET | `/categories` | Authenticated | List the five categories ordered by name | 200; array of category representations |

The category labels are the fixed product values listed in [`data-model.md`](data-model.md). Each category representation contains `id` and `name`.

### 3.4 Books

| Method | Path | Access | Purpose | Success |
|---|---|---|---|---|
| GET | `/books` | Authenticated | Search by `q` and `category_id`; paginate with `page` and `size` | 200; paginated book results |
| GET | `/books/{id}` | Authenticated | Get one active book | 200; book representation |
| POST | `/books` | `admin` | Create a book | 201; created book representation |
| PUT | `/books/{id}` | `admin` | Replace title, author, category, and total copies | 200; updated book representation |
| DELETE | `/books/{id}` | `admin` | Soft-delete a book | 204; no response body |
| PUT | `/books/{id}/cover` | `admin` | Upload a cover using `multipart/form-data`, field `file` | 204; no response body |
| GET | `/books/{id}/cover` | Authenticated | Retrieve the cover binary; 404 if no cover exists | 200 with image bytes |
| DELETE | `/books/{id}/cover` | `admin` | Remove a cover | 204; no response body |

Representative book representation:

```json
{
  "id": "…",
  "title": "Cien años de soledad",
  "author": "Gabriel García Márquez",
  "isbn": "9780307474728",
  "category": { "id": "…", "name": "Novela" },
  "total_copies": 4,
  "available_copies": 3,
  "cover_key": null
}
```

`POST /books` accepts `title`, `author`, `isbn`, `category_id`, and `total_copies`; the server initializes `available_copies` to the total. `cover_key` is opaque and is not a filesystem path. Cover uploads accept JPEG or PNG content up to 2 MB; validate the file content, not only its extension. Invalid uploads return 422.

`PUT /books/{id}` accepts `title`, `author`, `category_id`, and `total_copies`; the response uses the representative book representation above.

### 3.5 Members

| Method | Path | Access | Purpose | Success |
|---|---|---|---|---|
| GET | `/members` | `admin`, `librarian` | List/search by `q`, with `page` and `size` | 200; paginated member results |
| GET | `/members/{id}` | `admin`, `librarian` | Get member details and `active_loans` count | 200; member representation |
| POST | `/members` | `admin`, `librarian` | Create with `full_name` and `document_number` | 201; created member representation |

`active_loans` is the count of unreturned loan items. Member details must not be exposed through public endpoints. Member representations contain `id`, `full_name`, and `document_number`; the detail response also contains `active_loans`.

### 3.6 Loans

| Method | Path | Access | Purpose | Success |
|---|---|---|---|---|
| POST | `/loans` | `admin`, `librarian` | Record a loan | 201; returns the loan and its items |
| GET | `/loans` | `admin`, `librarian` | Filter by `from`, `to`, `status`, and `member_id`; paginate | 200; paginated loan results |
| GET | `/loans/{id}` | `admin`, `librarian` | Get loan details and calculated status | 200; loan representation |
| POST | `/loans/{loan_id}/items/{item_id}/return` | `admin`, `librarian` | Return one item | 200; returned item |
| GET | `/loans/overdue` | `admin`, `librarian` | List overdue items | 200; overdue-item results |

`POST /loans` request:

```json
{ "member_id": "…", "items": [{ "book_id": "…" }, { "book_id": "…" }] }
```

Representative 201 response:

```json
{
  "id": "…",
  "member_id": "…",
  "loaned_at": "2026-10-14T14:15:00Z",
  "status": "open",
  "items": [
    {
      "id": "…",
      "book_id": "…",
      "book_title": "Cien años de soledad",
      "isbn": "9780307474728",
      "category_name": "Novela",
      "due_at": "2026-10-28T14:15:00Z",
      "returned_at": null
    }
  ]
}
```

The calculated status is `open` when there are unreturned items and none is overdue, `overdue` when at least one unreturned item is overdue, and `closed` when all items have been returned. The return endpoint responds with 200 and the item with `returned_at` set.

Each overdue result contains `loan_id`, `item_id`, `book_title`, `member_name`, `due_at`, and `days_late`. Member names are included in this operational list; they are excluded from the most-borrowed report.

### 3.7 Most-borrowed report

| Method | Path | Access | Purpose | Success |
|---|---|---|---|---|
| GET | `/reports/most-borrowed` | `admin` | Require inclusive `from` and `to` dates | 200; report rows |

```json
{
  "from": "2026-10-01",
  "to": "2026-10-31",
  "rows": [
    { "book_id": "…", "book_title": "Cien años de soledad", "category_name": "Novela", "times_borrowed": 12 }
  ]
}
```

Group by `book_id`, snapshotted `book_title`, and snapshotted `category_name`. Order by `times_borrowed` descending, then `book_title` ascending. Do not include member or librarian details. The report is calculated from loan-item snapshots as specified in [ADR-004](adr/adr-004-aggregated-report.md).

## 4. Security and cross-origin access

| Concern | Rule |
|---|---|
| Token | Signed JWT, valid for 8 hours, with `sub` and `role`; no personal data in claims |
| Passwords | Store only a password hash; never include the hash in responses |
| Authorization | Check the role on every protected endpoint; a 403 response must not reveal whether a resource exists |
| CORS | Allow only the frontend origin configured for the environment |
| Cover uploads | Accept only JPEG and PNG content up to 2 MB; validate by file content, not filename extension |

## 5. Items pending approval

The report date interpretation for loans created with a backdated `loaned_at` remains to be resolved as described in [ADR-004](adr/adr-004-aggregated-report.md).
