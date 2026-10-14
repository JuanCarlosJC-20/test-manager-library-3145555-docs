# Manager Library Specification (Laravel)

**Status: Draft.** This folder is the planned specification for the React and Laravel implementation. It is not ready to guide implementation until its open decisions and cross-document checks are complete.

The existing `spec-.net/` and `spec-python/` folders are technical references for the same product. Their business requirements may inform this specification, but their framework-specific decisions are not automatically Laravel requirements.

## Reading order

1. [`constitution.md`](constitution.md) — project principles and naming conventions.
2. [`spec.md`](spec.md) — actors, scope, user stories, and business rules.
3. [`data-model.md`](data-model.md) and [`api-contract.md`](api-contract.md) — persistence model and HTTP interface.
4. [`architecture.md`](architecture.md), [`architecture-overview.md`](architecture-overview.md), and [`plan.md`](plan.md) — Laravel application structure and technical decisions.
5. [`tasks.md`](tasks.md) — work items and completion evidence organized by user story.
6. [`adr/`](adr/) — architecture decisions and their consequences.

## Readiness rule

Keep this specification in draft while decisions are unresolved or documents disagree. Start implementation only after the user stories, data model, API contract, architecture, plan, tasks, and ADRs have been reviewed against one another.
