# Product Specification — Manager Library

**Status: Ready for implementation.** This document defines product behavior for the React and Laravel implementation. Framework-specific implementation choices belong in `architecture.md` and `plan.md`.

The project is governed by [`constitution.md`](constitution.md). The data model is defined in [`data-model.md`](data-model.md), and the HTTP interface is defined in [`api-contract.md`](api-contract.md).

## 1. Actors

| Actor | Responsibilities | Authentication |
|---|---|---|
| **Administrator** (`admin`) | Manages the book catalog and members, creates librarian accounts, and views all reports | Username and password; receives a token |
| **Librarian** (`librarian`) | Records loans and returns; views the catalog, members, and overdue items | Username and password; receives a token |
| **Member** | Borrows books. A member does not sign in and is a record, not a system user | None |

The deployment provisions the `admin` role from environment variables. The role cannot be granted at runtime through the API.

## 2. Scope

**In scope:** a book catalog with optional covers, five fixed categories, members, multi-item loans, item-level returns, overdue items, a most-borrowed report, and role-based sign-in.

**Out of scope:** fines and payments, reservations, renewals, email or SMS, bulk import, inter-branch loans, and book attributes beyond those listed in this specification (no publisher, publication year, or language).

## 3. User stories and acceptance criteria

### HU-01 — Sign in

As a user, I want to authenticate so that I can perform actions allowed by my role.

- **CA-01.1** Valid credentials return a token that is valid for 8 hours.
- **CA-01.2** Invalid credentials return HTTP 401 with a message that does not reveal whether the username or password was incorrect.
- **CA-01.3** Every endpoint except sign-in and `/health` requires a token. A missing token returns 401; an insufficient role returns 403.

### HU-02 — Create librarian accounts (admin)

- **CA-02.1** An administrator creates a user with a username, password, and the `librarian` role.
- **CA-02.2** Usernames are trimmed, converted to lowercase, and unique. A duplicate returns 409.
- **CA-02.3** Requesting the `admin` role through the API returns 422. Passwords are never returned or written to logs.

### HU-03 — Manage books (admin)

- **CA-03.1** An administrator creates a book with a title, author, ISBN, category, and total copy count. Available copies initially equal the total copy count.
- **CA-03.2** An ISBN contains 13 digits and is unique among active books. A duplicate returns 409.
- **CA-03.3** Total copies must be at least 1. When the total changes, available copies change by the same difference. Return 422 if the requested total is below 1 or if the adjustment would make available copies negative.
- **CA-03.4** A cover upload accepts JPG or PNG files up to 2 MB and stores an opaque key. Without a cover, the field is `null`, never an empty string.
- **CA-03.5** Removing a book is a soft delete: it disappears from the catalog but remains available in past loan history. A book with open loans cannot be removed; the request returns 409.

### HU-04 — Search books

- **CA-04.1** Search supports partial text matching on title or author, category filtering, active books only, title ordering, and pagination with `page` and `size` (`size` has a maximum of 50).
- **CA-04.2** The response includes the total number of matching results.
- **CA-04.3** Looking up a nonexistent or soft-deleted book by ID returns 404.

### HU-05 — Manage members

- **CA-05.1** Administrators and librarians can create members with a name and document number. Document numbers are unique; a duplicate returns 409.
- **CA-05.2** Members can be listed and searched by name or document number, with pagination.
- **CA-05.3** Members cannot be deleted.

### HU-06 — Record a loan

- **CA-06.1** A loan belongs to exactly one member, is recorded by the authenticated user, and contains 1 to 3 items.
- **CA-06.2** A book cannot appear more than once in the same loan; a duplicate returns 422.
- **CA-06.3** Each loan item deducts one available copy as part of the same operation. If a copy is unavailable, the request returns 409 and creates no partial loan.
- **CA-06.4** A member cannot have more than 3 unreturned books across all loans; the request returns 422 and includes the current count.
- **CA-06.5** A member with any overdue item cannot borrow another book until all overdue items are returned; the request returns 409.
- **CA-06.6** The due date is `loaned_at + 14 days`. Each loan item snapshots the book title, ISBN, and category name at the time the loan is recorded.
- **CA-06.7** A recorded loan cannot be edited or deleted.

### HU-07 — Record a return

- **CA-07.1** Items are returned one at a time. Returning an item sets `returned_at` and adds one to the book's available copy count.
- **CA-07.2** Returning an already returned item returns 409 and never increments the available copy count a second time.
- **CA-07.3** Available copies must never exceed total copies; the database enforces this invariant.
- **CA-07.4** A loan is considered closed when all its items have been returned.

### HU-08 — View and list loans

- **CA-08.1** Viewing a loan returns its items and its calculated status: `open`, `overdue`, or `closed`.
- **CA-08.2** Loans can be listed by date range, status, and member, ordered by descending loan date, with pagination.
- **CA-08.3** Loan status is calculated and is not stored.

### HU-09 — View overdue items

- **CA-09.1** List unreturned items whose `due_at` is earlier than the current time. Each result includes the book, member, and number of days overdue.
- **CA-09.2** Results are ordered by descending days overdue. Only `admin` and `librarian` roles can access this list.

### HU-10 — View the most-borrowed report

- **CA-10.1** For a date range, return one row per `book_id`, snapshotted `book_title`, and snapshotted `category_name`, with the number of times borrowed. Order by `times_borrowed` descending, then by `book_title` ascending.
- **CA-10.2** If the end date is earlier than the start date, return 422. A range with no loans returns an empty list, not an error.
- **CA-10.3** The report is not grouped by librarian or member.
- **CA-10.4** A reporting period is a requested date range whose end date has passed. Renaming a book or its category after that period does not change the report values for that range; the report uses the title and category snapshots stored on loan items.
- **CA-10.5** Only the `admin` role can access the report.

## 4. Business rules

| ID | Rule |
|---|---|
| RN-1 | Available copies are always between zero and total copies, inclusive. |
| RN-2 | The loan period is 14 days. A member may have at most 3 unreturned books. |
| RN-3 | A member with overdue items cannot borrow more books until those items are returned. |
| RN-4 | Values snapshotted on a loan item (title, ISBN, and category) do not change when the catalog changes. |
| RN-5 | Loans and returns are not deleted. Books are soft-deleted. |
| RN-6 | All timestamps are UTC instants. User-facing messages are in Spanish; code identifiers and database columns use English. |

## 5. Non-functional requirements

| ID | Requirement | Verification |
|---|---|---|
| RNF-1 | Book search responds in under 300 ms with 5,000 books. | Load test using the `tool` seeder. |
| RNF-2 | When two librarians attempt to loan the last copy at the same time, one succeeds and the other receives 409. | Concurrent request test. |
| RNF-3 | Passwords are never transmitted or stored in plaintext and never appear in logs. | Review logs and responses. |
| RNF-4 | All HTTP errors use the same response body. | API contract test. |
| RNF-5 | The system starts with one command, and the database starts empty. | Verify against the `infra` README. |
