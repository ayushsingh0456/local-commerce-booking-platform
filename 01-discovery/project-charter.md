# Project Charter: Quartier Release 1

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DIS-CH |
| Version | 1.1 |
| Status | Approved |
| Owner | Product Owner (prepared by the Business Analyst) |
| Last updated | 2026-03-02 |
| Reviewers | Managing Director (sponsor); Engineering Lead; Partner Operations Lead; Compliance Lead; Finance Lead |

**Purpose and scope.** Authorises the Quartier Release 1 project. It states why the project exists, what it must deliver, by when, with whom and within which limits. Detailed requirements are in the [BRD](../02-requirements/BRD.md) and the [SRS](../02-requirements/SRS.md).

## 1. Vision

> For independent local businesses in Vienna, and the neighbors who use them, Quartier is one app and one portal to order, book and run the day. It replaces the phone, the paper diary and the printed allergen sheet. It shows every business with its own look, ranks them neutrally and gives complete allergen information.

## 2. Business case in brief

Discovery (2026-01-12 to 2026-02-20) with 14 businesses, 22 customer interviews and a survey of 410 residents found:
- uncollected phone orders (6.8%);
- salon no-shows (9.5%);
- unanswered booking calls (38%);
- incomplete allergen information (58% of items complete);
- slow set-up on existing tools (11 business days).

Quartier earns an 8% commission on in-app paid food orders. The [BRD section 11](../02-requirements/BRD.md#11-business-model-and-benefits) gives the illustrative economics.

## 3. Objectives

The six measurable objectives OBJ-01 to OBJ-06, with baselines and targets, are in [BRD section 4](../02-requirements/BRD.md#4-business-objectives). They are measured over Q4 2026.

## 4. Scope

| In scope (R1) | Out of scope |
|---|---|
| Customer Mobile App (iOS, Android) for discovery, pickup orders and appointments | Delivery; table reservations |
| Business Web Portal with POS view | Counter payments and cash register functions |
| Platform Admin portal for onboarding, verification and lifecycle | Integrations with existing cash registers or booking tools |
| In-app card and wallet payments with 3-D Secure for food | Online payment for appointments |
| German and English | Other languages |
| Vienna | Other cities |

## 5. Milestones

| Milestone | Target date |
|---|---|
| Discovery complete, BRD v1.0 | 2026-02-27 |
| SRS v1.0 baseline; Sprint 1 starts | 2026-03-02 / 2026-03-09 |
| Build complete (7 sprints) | 2026-06-12 |
| UAT sign-off | 2026-06-26 |
| Pilot go-live in Neubau and Mariahilf | 2026-07-01 |
| Vienna launch | 2026-09-01 |

## 6. Team and governance

| Role | Responsibility |
|---|---|
| Managing Director (sponsor) | Funding; final scope arbitration |
| Product Owner | Backlog, priorities, acceptance; chairs the change control board |
| Business Analyst | Discovery, BRD, SRS, business rules, stories, traceability, UAT facilitation, post-incident reviews |
| Engineering Lead | Architecture, delivery, operations |
| Developers (2 backend, 2 Flutter, 1 web) | Build |
| QA Lead and QA engineer | Test strategy, automation, regression |
| UX Designer | Research support, design, accessibility |
| Partner Operations Lead | Business onboarding and verification process |
| Partner Success Lead | Pilot businesses, training, communication |
| Compliance Lead (with external data protection officer and legal counsel) | Food information, GDPR, consumer and platform rules |
| Finance Lead | VAT, commission, payouts, reconciliation |

Governance: a fortnightly steering meeting (sponsor, Product Owner, Engineering Lead, BA). Change control follows the [CR log](../05-delivery/change-request-log.md). The project tracks risks in the [RAID log](../05-delivery/raid-log.md).

## 7. Budget and constraints (illustrative)

| Item | Amount |
|---|---|
| Team cost, build phase (14 weeks plus hardening) | EUR 410,000 |
| Cloud, card payment provider set-up, tools | EUR 28,000 |
| External penetration test, legal and data protection review | EUR 36,000 |
| Contingency (10%) | EUR 47,400 |
| **Total** | **EUR 521,400** |

Constraints: EU hosting only; German first; one team; the pilot date of 2026-07-01 is fixed because pilot businesses planned their summer around it.

## 8. Initial risks

The top risks at charter time were:
- allergen declarations wrong or incomplete;
- adoption by counter staff;
- payment provider onboarding for small businesses;
- the cash register question.

All four are tracked in the RAID log (R-01, R-03, R-05, R-06).

## 9. Success criteria

The project succeeds when R1 is live in Vienna, OBJ-01 to OBJ-06 meet their targets over Q4 2026 (or have agreed corrective plans), and no SEV-1 incident remains without a completed CAPA.

## 10. Approval

| Role | Decision | Date |
|---|---|---|
| Managing Director (sponsor) | Approved | 2026-03-02 |
| Product Owner | Approved | 2026-03-02 |
| Engineering Lead | Approved | 2026-03-02 |
| Compliance Lead | Approved | 2026-03-02 |

## Related documents

- [Business Requirements Document](../02-requirements/BRD.md)
- [Stakeholder register and RACI](stakeholder-register-raci.md)
- [Personas](personas.md)
- [Current vs future state](current-vs-future-state.md)
- [Market and competitor scan](market-and-competitor-scan.md)
- [Release and sprint plan](../05-delivery/release-and-sprint-plan.md)
