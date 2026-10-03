# ADR-001: One backend for three apps, built as a modular monolith

## Document control

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-02-18 |
| Deciders | Engineering Lead, Product Owner, Business Analyst |
| Consulted | Backend developers, mobile developers, QA Lead |
| Related requirements | NFR-SEC-05, NFR-MNT-01, NFR-CAP-01, FR-POS-07 |
| Supersedes | None |

## Context and problem statement

Quartier has three front ends: the Customer Mobile App, the Business Web Portal with its POS view, and the Platform Admin portal. They all read and write the same orders, appointments, catalog and businesses. A customer's order must appear on the order board within 3 seconds (NFR-PERF-04). An allergen change confirmed in the portal must reach the app within 60 seconds (NFR-DAT-02). The team is one cross-functional squad with 2 backend developers.

How do we structure the backend so that the three apps share one truth? It also has to keep the roles apart and stay small enough to run.

## Decision drivers

- One consistent data set: an order is one record, whoever looks at it.
- Strict separation of what each role can do (BR-039) and of each business's data (NFR-SEC-05).
- Small team: one deployable, one database, few moving parts.
- Rules must be testable in one place (NFR-MNT-01).
- Room to split later if one area grows much faster than the others.

## Considered options

1. **Microservices per domain** (ordering, appointments, catalog, payments, onboarding), each with its own database.
2. **A backend-for-frontend per app** in front of a shared core service.
3. **One modular monolith** with internal modules per SRS module, one PostgreSQL database and role-scoped route groups.

## Decision outcome

Chosen option: **3, one modular monolith.** NestJS modules mirror the SRS modules (accounts, discovery, catalog, ordering, payments, appointments, pos, onboarding), plus notifications and audit. Modules talk through explicit service interfaces and domain events, never through each other's tables.

Route groups separate the surfaces:

| Route group | Callers | Guard |
|---|---|---|
| `/v1` | Customer Mobile App (guests on public reads) | Customer token, or none for public reads |
| `/v1/business` | Business Web Portal and POS view | Portal token with role Business Admin or Staff; Staff limited to POS operations (FR-POS-07) |
| `/v1/platform` | Platform Admin portal | Platform token with second factor (NFR-SEC-04) |

Every business-scoped table carries `business_id`. Each transaction sets `app.business_id` from the token, and PostgreSQL row-level security policies enforce it. A missing filter in application code returns no rows rather than another business's rows.

### Positive consequences

- One transaction can create an order, decrement slot capacity and write the audit event. Distributed sagas are not needed for the core flows.
- Business rules live in one code base, so the rule-coverage gate (NFR-MNT-01) has one place to measure.
- One deployment pipeline. Blue/green releases are simple.

### Negative consequences

- A defect in one module can affect all three apps. Mitigated by module boundaries enforced in CI (no cross-module table access) and by the error monitoring per route group.
- Scaling is coarse: the whole API scales together. Acceptable at the Vienna volumes in NFR-CAP-01. The realtime gateway and workers are separate processes from the same code base and scale on their own.

## Pros and cons of the other options

| Option | Pros | Cons |
|---|---|---|
| 1 Microservices | Independent scaling and deployment | Distributed transactions for order plus capacity plus payment; far more operations work than a team of 2 backend developers can carry; data consistency for the order board becomes eventual |
| 2 BFF per app | Each front end gets a tailored API | Three extra services that mostly pass calls through; role rules risk being duplicated in each BFF |

## Validation

- The isolation test suite (TC-NFR-006) runs each Business operation with a token for another business and expects 404 and an audit log entry.
- The architecture fitness check in CI fails a build when one module imports another module's repository.

## Links

- [System architecture](../system-architecture.md)
- [ADR-002](ADR-002-slot-holds-redis-database-guard.md)
- [ADR-003](ADR-003-marketplace-payments-authorize-capture.md)
- [API guidelines](../../../04-api/api-guidelines.md)

## Related documents

- [Software Requirements Specification](../../../02-requirements/SRS.md)
- [Non-functional requirements](../../../02-requirements/non-functional-requirements.md)
