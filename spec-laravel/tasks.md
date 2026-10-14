# Implementation Tasks — Manager Library

**Status: Ready for implementation.** These are implementation planning tasks only; no application code is part of this documentation work.

## 1. Documentation readiness gate

- [x] Resolve the decisions in [`plan.md`](plan.md).
- [x] Reconcile every user story with [`data-model.md`](data-model.md) and [`api-contract.md`](api-contract.md).
- [x] Reconcile [`architecture.md`](architecture.md), all ADRs, and task evidence with the Laravel target.
- [x] Confirm all explanatory documentation is English and follows [`constitution.md`](constitution.md); preserve Spanish product labels and user-facing messages where required.

## 2. User-story implementation map

| User story | Implementation work | Dependencies | Completion evidence | Status |
|---|---|---|---|---|
| HU-01 — Sign in | Implement credential validation, JWT issue/expiry, authentication middleware, and generic invalid-credential errors | D-01, D-04, schema | Valid login returns an 8-hour token; invalid login returns 401 without revealing which credential failed | Not started |
| HU-02 — Create librarian accounts | Add admin-only account creation, username normalization/uniqueness, and bcrypt hashing | HU-01, D-04, schema | Admin can create a librarian; duplicate username returns 409; plaintext password and hash never appear in responses or logs | Not started |
| HU-03 — Manage books | Add create/update/soft-delete flows, active ISBN uniqueness, copy adjustments, and cover upload/deletion | D-02, D-05, ADR-003 | Acceptance criteria pass, including open-loan conflict, ISBN reuse after soft deletion, and approved response codes | Not started |
| HU-04 — Search books | Add active catalog search by title/author and category, ordering, pagination, and detail lookup | HU-03, schema | Search, pagination count, ordering, and 404 behavior meet CA-04.1–CA-04.3 | Not started |
| HU-05 — Manage members | Add member create, list, search, detail, and active-loan count | D-02, schema | Unique document number is enforced; only authorized roles access member data | Not started |
| HU-06 — Record a loan | Add loan validation, member restrictions, member/book row locks, atomic copy updates, loan and snapshot-item creation | HU-01, HU-05, HU-03, D-03 | All CA-06 criteria pass; concurrent last-copy requests yield one 201 and one 409; concurrent loans cannot exceed a member's three-item limit; no partial writes | Not started |
| HU-07 — Record a return | Add item-level return and atomic available-copy increment | HU-06, D-03 | Return sets `returned_at` once; repeated return returns 409 and does not increment copies again | Not started |
| HU-08 — View and list loans | Add loan details with calculated status and filtered, ordered, paginated list | HU-06, HU-07 | Status reflects all item return/due-date states; filters and ordering meet CA-08.1–CA-08.3 | Not started |
| HU-09 — View overdue items | Add overdue-item query with member/book details and descending days-late ordering | HU-06, D-02 | Only unreturned overdue items appear, ordered as required; access is limited to admin and librarian | Not started |
| HU-10 — View most-borrowed report | Add inclusive date-range validation and aggregate query over loan-item snapshots | HU-06, D-02, ADR-004 | Grouping, ordering, empty range, date validation, privacy, and admin-only access meet CA-10.1–CA-10.5 | Not started |

## 3. Cross-cutting verification

- [ ] Apply all migrations to an empty PostgreSQL 16 database and verify seeded categories.
- [ ] Verify API error responses use the single envelope and documented error codes.
- [ ] Verify all protected routes enforce authentication and role authorization.
- [ ] Verify no endpoint exposes a password, password hash, stack trace, SQL, or server path.
- [ ] Run the agreed test suite and record reproducible commands and outcomes in the API repository.
