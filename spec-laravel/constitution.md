# Project Constitution

**Status: Draft.** This document will define the principles that every other document in this specification must follow.

## Documentation language

All prose, headings, examples, and explanatory labels in this specification must be written in English. Product data examples must also be reviewed for language consistency.

## Naming conventions

The following conventions must be agreed and then applied consistently:

| Area | Convention | Status |
|---|---|---|
| Database table names | To be decided | Open |
| Database column names | To be decided | Open |
| JSON field names | To be decided | Open |
| HTTP collection routes | To be decided | Open |
| PHP classes and methods | To be decided | Open |
| Entity prefixes | To be decided | Open |

## Sources of truth

- `spec.md` owns product behavior and acceptance criteria.
- `data-model.md` owns persistent entities, columns, constraints, and indexes.
- `api-contract.md` owns HTTP routes, request and response fields, and error formats.
- `architecture.md` and `plan.md` own implementation guidance for the Laravel target.

These ownership rules and all remaining principles require review before this constitution is approved.
