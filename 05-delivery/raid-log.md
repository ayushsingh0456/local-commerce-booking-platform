# RAID Log: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DEL-RAID |
| Version | 1.6 |
| Status | Active |
| Owner | Business Analyst (maintained with the Product Owner) |
| Last updated | 2026-10-02 |
| Reviewers | Engineering Lead; Partner Operations Lead; Compliance Lead |

**Purpose and scope.** Risks, assumptions, issues and dependencies for R1 and R1.1, reviewed weekly in refinement and before each CCB. Scores: likelihood and impact from 1 (low) to 5 (high); exposure = likelihood x impact.

## 1. Risks

| ID | Risk | L | I | Exposure | Response | Owner | Status | Review |
|---|---|---|---|---|---|---|---|---|
| R-01 | Businesses declare allergens incorrectly for their own recipes | 3 | 5 | 15 | Gate and confirmation (BR-011); change control (BR-013); onboarding guide with the official allergen guidance for food businesses; sample audit of 10 items per new business | Compliance Lead | Open (monitored) | 2026-10-15 |
| R-02 | Slow 3-D Secure challenges break holds at lunch | 4 | 3 | 12 | Materialised as INC-2026-011; mitigated by CR-006 and NFR-OBS-02 alerts | Engineering Lead | Mitigated in R1.1 | 2026-10-15 |
| R-03 | Counter staff miss new orders in noise | 3 | 4 | 12 | Repeating sound, flashing border, countdown (BR-040); UAT-05 lunch simulation | UX Designer | Closed after UAT | Closed 2026-06-26 |
| R-04 | Too few businesses in an area for customers to return | 3 | 4 | 12 | Pilot concentrated in two districts; Partner Success targets 60 Live locations in Neubau and Mariahilf before wider marketing | Partner Success Lead | Open | 2026-11-01 |
| R-05 | Card payment provider onboarding rejects or delays small businesses | 2 | 4 | 8 | Pre-check of documents in verification; salons skip payment onboarding in R1 | Partner Operations Lead | Open | 2026-10-15 |
| R-06 | Tax authority view on prepaid app orders requires register receipts (TBD-01) | 2 | 4 | 8 | Written opinion requested; receipts already in the business's name; design keeps the business as seller | Compliance Lead | Open | 2026-11-30 |
| R-07 | Stale data served from caches | 3 | 5 | 15 | Materialised as INC-2026-006; mitigated by CR-005 and NFR-DAT-02 | Engineering Lead | Mitigated | 2026-10-15 |
| R-08 | App store review delays the R1.1 app release | 2 | 2 | 4 | Submit 5 working days before 2026-10-07; server-side hotfixes already cover the safety part | Engineering Lead | Open | 2026-10-05 |
| R-09 | Growth ends the micro-enterprise exemptions (accessibility act, DSA) earlier than planned | 2 | 3 | 6 | WCAG 2.2 AA already targeted; onboarding data covers DSA Art. 30 except ID copies (TBD-10) | Compliance Lead | Open | 2027-01-15 |

## 2. Assumptions

| ID | Assumption | Validated by | Status |
|---|---|---|---|
| A-01 | The business is the seller; the platform is an intermediary for VAT and consumer law | External legal counsel, 2026-02-20 | Confirmed |
| A-02 | The right of withdrawal does not apply to perishable food prepared to order or to dated appointments | External legal counsel, 2026-02-20 | Confirmed |
| A-03 | Customers in the pilot districts have smartphones with cards enabled for 3-D Secure or wallets | Discovery survey: 96% of 410 respondents | Confirmed |
| A-04 | Pilot businesses keep their certified cash registers | Pilot agreements | Confirmed |
| A-05 | Straight-line distance is good enough for sorting in a dense city | Usability sessions in S3 | Confirmed for R1 |
| A-06 | Austrian VAT rates of 10% and 20% apply to the pilot categories at launch | Finance Lead with the businesses' tax advisers | Confirmed; rates are effective-dated data (NFR-MNT-02) |

## 3. Issues

| ID | Issue | Impact | Action | Owner | Status |
|---|---|---|---|---|---|
| I-01 | Card payment provider sandbox lacked a slow-challenge simulation | Could not reproduce INC-2026-011 in staging | Provider added a configurable delay card; TC-ORD-010 now uses it | Engineering Lead | Closed 2026-09-15 |
| I-02 | DEF-118: Android cold start 3.4 s p90 against a 3.0 s target | NFR-PERF-02 not met on the reference device | Image decoding moved off the main thread; fix planned for R1.2 | Engineering Lead | Open |
| I-03 | DEF-121: auto-pause email sent twice when two board tabs disconnect | Business Admin receives a duplicate email | Deduplicate by pause event; accepted for R1.1 with a fix in R1.2 | Engineering Lead | Open |
| I-04 | Allergen filter clarification ran into a second session | US-010 AC4 and AC5 moved from S3 to S5 | Spec 001 clarification session 2 held on 2026-04-15 | Business Analyst | Closed 2026-04-15 |

## 4. Dependencies

| ID | Dependency | Needed for | Status |
|---|---|---|---|
| D-01 | Marketplace contract with the card payment provider | EP-03, FR-ONB-03 | Signed 2026-03-02 |
| D-02 | Maps and geocoding provider contract (EU processing) | FR-CAT-02, BR-006 | Signed 2026-03-05 |
| D-03 | Apple and Google developer accounts; Sign in with Apple configuration | FR-ACC-03 | Done 2026-03-13 |
| D-04 | Pilot business availability for UAT (14 businesses, 2 weeks) | UAT plan | Done 2026-06-26 |
| D-05 | External penetration test slot | NFR-SEC-01 | Done 2026-06-08 (R1); 2026-09-29 (R1.1) |
| D-06 | Tax counsel opinion on register receipts | TBD-01 | Requested 2026-09-02 |

## Related documents

- [Release and sprint plan](release-and-sprint-plan.md)
- [Decision log](decision-log.md)
- [Change request log](change-request-log.md)
- [Software Requirements Specification](../02-requirements/SRS.md)
- [Incident management process](../07-operations/incident-management-process.md)
