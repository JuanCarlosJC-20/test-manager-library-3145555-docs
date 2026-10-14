# ADR-003: Soft-Delete Books and Reuse ISBNs Only After Deletion

- **Status:** Accepted
- **Date:** 2026-10-14
- **Context:** Removing a book must hide it from the active catalog without losing historical loan records. The product requirements reject removal while the book has open loan items. The ISBN uniqueness rule applies to active books.
- **Decision:** Soft-delete books by setting `books.deleted_at`. Catalog search and ID lookup expose active books only. Keep historical loan-item snapshots and foreign keys intact. Enforce ISBN uniqueness with a PostgreSQL partial unique index over rows where `deleted_at IS NULL`; an ISBN may be reused after the previous book is soft-deleted.
- **Alternatives considered:** Physical deletion would conflict with retained loan history and the restrictive foreign key. Retaining a unique ISBN across deleted records would prevent reuse even though the product requirement limits uniqueness to active books.
- **Consequences:** Book deletion is an update, not a physical delete. Past loan-item snapshots remain reportable. Restore behavior is not part of the product scope; any future restore feature must define how to handle an ISBN that has since been reused.
- **Verification:** An active book's ISBN cannot be assigned to another active book. After soft deletion, a new active book may use that ISBN. Soft-deleting a book with open loan items returns 409; historical loan records remain readable.
