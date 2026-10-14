# ADR-004: Calculate the Most-Borrowed Report from Loan-Item Snapshots

- **Status:** Accepted
- **Context:** The most-borrowed report (HU-10) covers a requested date range. Reading current catalog values would allow later book or category renames to change report values for loans already recorded.
- **Decision:** Do not persist report rows. Calculate the report with one aggregated read query, grouping by `book_id`, snapshotted `book_title`, and snapshotted `category_name` from each loan item. `loan_item.category_name` has no foreign key by design. If a book has different snapshotted titles or category names within the range, return separate rows for those distinct snapshot groups. Order by `times_borrowed` descending, then `book_title` ascending.
- **Alternatives considered:** Joining to the live catalog would make historical report values change after catalog edits. Grouping by only the current catalog values would hide historical differences. Persisting report rows would create a second source of truth.
- **Consequences:** CA-10.1 groups by the book ID and the title and category snapshots, not by the book ID alone. Later catalog renames do not change the values stored in existing loan items or the report for those loans.
- **Verification:** Record a loan, rename its book or category, then request a range containing that loan. The report retains the title and category values captured on the loan item.
- **Open scope clarification:** This decision covers catalog renames. It does not specify whether a loan may be entered later with a `loaned_at` inside a past reporting range; no additional restriction is introduced here.
