# Manager Library Specification (Laravel)

**Status: Ready for implementation.** This folder contains the reviewed product and technical specification for the React and Laravel implementation. The decisions that were open during drafting are recorded in the accepted ADRs and technical plan.

The existing `spec-.net/` and `spec-python/` folders are technical references for the same product. Their business requirements may inform this specification, but their framework-specific decisions are not automatically Laravel requirements.

## Reading order

1. [`constitution.md`](constitution.md) — project principles and naming conventions.
2. [`spec.md`](spec.md) — actors, scope, user stories, and business rules.
3. [`data-model.md`](data-model.md) and [`api-contract.md`](api-contract.md) — persistence model and HTTP interface.
4. [`architecture.md`](architecture.md), [`architecture-overview.md`](architecture-overview.md), and [`plan.md`](plan.md) — Laravel application structure and technical decisions.
5. [`tasks.md`](tasks.md) — work items and completion evidence organized by user story.
6. [`adr/`](adr/) — architecture decisions and their consequences.

## Readiness rule

Before implementation, confirm that this specification remains consistent with the current project repositories. Any change to product behavior or an accepted architectural decision must be reviewed and recorded in the owning document and ADR.
