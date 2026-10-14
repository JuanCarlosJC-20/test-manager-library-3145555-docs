# ADR-001: Laravel Migrations Own the Database Schema

- **Status:** Accepted
- **Date:** 2026-10-14
- **Context:** The database schema, constraints, indexes, and fixed category data must be reproducible and evolve alongside the Laravel application. The infrastructure repository provides an empty database and must not become a competing schema source.
- **Decision:** Version Laravel database migrations as the sole source of truth for all DDL. The initial migration creates the schema objects and inserts the five fixed category rows using their stable UUIDs. Infrastructure provisions only the empty database service and required runtime configuration; it does not create application tables or seed category rows.
- **Alternatives considered:** Manually maintained SQL scripts would separate schema changes from Laravel migrations. Putting DDL or reference data in the infrastructure repository would split ownership and risk drift.
- **Consequences:** Schema changes and database-enforced constraints are reviewed with the application changes that require them. The Laravel migration history must be sufficient to recreate the schema and its fixed category data from an empty database.
- **Verification:** Applying all migrations to an empty PostgreSQL 16 database produces the schema and five category rows defined in `data-model.md`. The infrastructure repository contains no application `CREATE TABLE` statements.
