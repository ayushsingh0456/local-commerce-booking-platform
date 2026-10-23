# Decision Log: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | LC-DEL-DEC |
| Version | 1.4 |
| Status | Active |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Product Owner; Engineering Lead |

**Purpose and scope.** Decisions that shaped the platform's scope and requirements, with the alternatives considered and who decided. Architecture decisions have their own ADRs and are cross-referenced here; change requests are in the [CR log](change-request-log.md).

| ID | Date | Decision | Alternatives considered | Rationale | Decided by | Affects |
|---|---|---|---|---|---|---|
| DEC-01 | 2026-02-10 | Food is pickup only in R1; no delivery | Delivery through a courier partner; own couriers | Delivery is a second operation (zones, fees, couriers) that pilot businesses do not run; lunch pickup is where the phone hurts most | Product Owner, Managing Director | BRD 5.3, BR-017 |
| DEC-02 | 2026-02-12 | Appointments are paid at the venue in R1 | Deposits; full prepayment | Salons asked for simplicity; skipping payment onboarding speeds up OBJ-05; no-shows addressed by reminders first | Product Owner | FR-APT-09, BR-032, BR-043 |
| DEC-03 | 2026-02-17 | An allergen declaration is a publishing gate; "information not available, ask the business" is not allowed for a published item | Show "not available" as discovery suggested; warn only | Distance sales need the information before purchase; a missing declaration is the riskiest state for a customer with an allergy | Product Owner, Compliance Lead | BR-011, FR-CAT-04, OBJ-04 |
| DEC-04 | 2026-02-18 | One backend as a modular monolith for three apps | Microservices; BFF per app | Consistency and team size | Engineering Lead | [ADR-001](../03-design/architecture/adr/ADR-001-one-backend-modular-monolith.md) |
| DEC-05 | 2026-02-19 | Neutral ranking by distance or name; no public ratings; feedback private to the platform | Ratings; promoted placement | Product principle; fairness to small businesses; less moderation and compliance work | Managing Director, Product Owner | BR-008, CR-007 |
| DEC-06 | 2026-02-24 | Age is a self-declaration of 14 or older, without a date of birth | Date of birth; no age question | Data minimization; Austrian age of consent 14; no age-restricted goods in R1 | Compliance Lead | BR-002 |
| DEC-07 | 2026-02-25 | Marketplace payments with connected accounts; capture at acceptance | Merchant of record; capture at placement or hand-over | No funds held by the platform; rejections cost nothing | Managing Director, Finance Lead, Engineering Lead | [ADR-003](../03-design/architecture/adr/ADR-003-marketplace-payments-authorize-capture.md) |
| DEC-08 | 2026-03-04 | Holds in Redis with TTL; the database decides | Holds as rows; Redis only | Self-expiring holds and guaranteed capacity | Engineering Lead | [ADR-002](../03-design/architecture/adr/ADR-002-slot-holds-redis-database-guard.md) |
| DEC-09 | 2026-03-11 | Staff use the Business Web Portal with a role-based POS view; no separate staff app | Separate tablet app | One login and one code base; the discovery input already described it this way | Product Owner, UX Designer | BR-039, FR-POS-07 |
| DEC-10 | 2026-04-15 | The allergen filter is stored only on the device | Store in the account with explicit consent | Avoids processing health data on the server in R1 | Compliance Lead | FR-DSC-09, [spec 001](../08-ai-assisted-ba/specs/001-allergen-filter-on-menus/spec.md) |
| DEC-11 | 2026-05-13 | The POS view is an order board, not a cash register | Full POS with counter payments | Cash register rules (CR-003 rejected) | Product Owner, Compliance Lead | BR-024, CR-003 |
| DEC-12 | 2026-06-10 | Sign-in uses one message for unknown email and wrong password; reset request does not reveal whether an account exists | Discovery wording "no account found" | Account enumeration risk; UAT participants did not miss the specific message | Product Owner, Engineering Lead | FR-ACC-03, FR-ACC-04 |
| DEC-13 | 2026-08-14 | Menu data is never cached by shared caches for more than 60 seconds | Keep 24-hour CDN caching for speed | Safety over speed for allergen data (INC-2026-006) | Engineering Lead, Product Owner | NFR-DAT-02, CR-005 |
| DEC-14 | 2026-09-11 | A hold is a courtesy, not a guarantee: capacity is re-checked at authorization | Longer holds only | Longer holds alone still fail during an issuer outage (INC-2026-011) | Engineering Lead, Product Owner | BR-020, CR-006 |

## Related documents

- [Change request log](change-request-log.md)
- [RAID log](raid-log.md)
- [Business Requirements Document](../02-requirements/BRD.md)
- [System architecture](../03-design/architecture/system-architecture.md)
