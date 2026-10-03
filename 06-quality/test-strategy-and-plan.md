# Test Strategy and Plan: Quartier

## Document control

| Field | Value |
|---|---|
| Document ID | QT-QA-01 |
| Version | 1.4 |
| Status | Approved |
| Owner | QA Lead (co-authored with the Business Analyst) |
| Last updated | 2026-10-02 |
| Reviewers | Engineering Lead; Product Owner; Compliance Lead; Finance Lead |

**Purpose and scope.** How Quartier R1 and R1.1 are tested: levels, techniques, environments, data, entry and exit criteria, defect handling and the current status. The structure follows the test plan and test strategy items of ISO/IEC/IEEE 29119-3. The test cases themselves are in [test-cases.md](test-cases.md) and [test-cases.csv](test-cases.csv); acceptance by the business is in [uat-plan-and-scripts.md](uat-plan-and-scripts.md).

## 1. Test items and risk-based focus

Testing effort follows product risk. The highest risks are those where an error harms a customer's health, takes their money, or breaks a promise to a business.

| Risk area | Why it is high | Main rules | Test emphasis |
|---|---|---|---|
| Allergen information | Health of customers with allergies | BR-010 to BR-013, NFR-DAT-02 | Decision tables per declaration state; freshness probe; version re-confirmation; filter privacy |
| Capacity under concurrency | Overloaded kitchens and double-booked stylists | BR-018 to BR-020, BR-026, BR-028 | Concurrency tests with parallel requests; slow 3-D Secure simulation; anomaly alert tests |
| Money | Wrong VAT, double charges, refunds above capture | BR-033 to BR-038 | Worked examples to the cent; webhook replay; reconciliation |
| Time | Cutoffs, timeouts, notice, horizon, daylight saving | BR-017, BR-022, BR-023, BR-026, BR-029, BR-030, NFR-DAT-01 | Boundary values with a virtual clock |
| Access | Staff seeing financial functions; cross-business leaks | BR-039, NFR-SEC-05 | Role matrix tests through UI and API; isolation suite |

## 2. Test levels

| Level | Scope | Owner | Automation | Environment |
|---|---|---|---|---|
| Unit | Rule functions (slot computation, VAT allocation, start times), validators | Developers | 100% automated; rule coverage gate (NFR-MNT-01) | CI |
| API contract | Every operation in `openapi.yaml`: schemas, error codes, idempotency | Developers, QA | Automated on every merge | CI and staging |
| Integration | Card payment provider sandbox (authorize, capture, void, refund, webhooks, hosted onboarding), FCM, email, geocoding, VAT number validation stub | QA | Automated nightly | Staging |
| System | End-to-end flows across the app, portal and API ([test cases](test-cases.md)) | QA, BA | 80% or more automated | Staging with virtual clock |
| Non-functional | Load, device lab, accessibility, security, chaos, restore | QA, Engineering Lead, external testers | Mixed | Staging, production probes |
| UAT | Business acceptance by pilot businesses and a customer panel | BA (facilitator), pilot businesses | Manual, scripted | Staging with UAT data |

## 3. Test design techniques

| Technique | Used for | Example |
|---|---|---|
| Boundary value analysis | Every time or amount threshold | 11:55 and 11:56 for the earliest slot (TC-ORD-002); € 14,99 and € 15,00 for MOHN3 (TC-PAY-004) |
| Decision table | Rules with combined conditions | Cancellation table 3.6a (TC-ORD-008); allergen display (TC-DSC-004) |
| State transition | Order, payment, appointment, hold and business lifecycles | SM-1 to SM-6 in [state machines](../03-design/diagrams/state-machines.md) |
| Equivalence partitioning | Inputs with classes of behavior | Voucher codes, roles, payment methods |
| Use case | End-to-end journeys | Order from cart to hand-over (TC-ORD-006) |
| Error guessing | Concurrency, duplicates, outages | Two confirmations at once (TC-APT-005); duplicate webhooks (TC-PAY-007) |
| Pairwise | Device and browser matrix | TC-NFR-016 |

## 4. Environments and data

- **Staging** mirrors production with synthetic data from [test-data](test-data/README.md), the card payment provider sandbox (including a configurable slow-challenge card since I-01), a VAT number validation stub and a **virtual clock** per test location, so that cutoffs and timeouts are tested to the second.
- **Device lab:** iPhone SE (3rd gen) and iPhone 15 on iOS 16 and 18; a 3-year-old mid-range Android phone (reference device for NFR-PERF-02) and a current one on Android 9 and 15; 10-inch Android and iPad tablets for the POS view.
- **No production personal data** is used in any test environment.

## 5. Entry and exit criteria

| Stage | Entry | Exit |
|---|---|---|
| System test per sprint | Stories meet the Definition of Ready; build deployed to staging | All P1 cases for the sprint's stories pass; no open Critical defect |
| Regression (release) | Code freeze; release candidate deployed | All P1 cases pass; P2 failures only with an accepted defect; rule coverage 100% |
| UAT | System test exit met; UAT data loaded; participants trained | Every UAT script passed or passed with an accepted Medium or Low defect; sign-off by each participating business and the customer panel |
| Release | Regression and UAT exit met | Release Definition of Done ([DoR and DoD](../05-delivery/definition-of-ready-and-done.md)) |

## 6. Defect management

| Severity | Definition | Response |
|---|---|---|
| Critical | Customer health or money at risk; data leak; no workaround | Fix before release; hotfix in production within 24 hours |
| High | Core flow blocked for a group of users; wrong amount without loss | Fix before release |
| Medium | Function impaired with a workaround | Product Owner decides; fix in the next release |
| Low | Cosmetic or copy | Backlog |

Every defect found by a failing test that the requirements did not anticipate is classified as a **requirement gap**. The BA then updates the rule or acceptance criteria, as with DEF-047 and DEF-066 below.

**Defect summary (selected).**

| Defect | Found in | Severity | Summary | Outcome |
|---|---|---|---|---|
| DEF-047 | S4 system test (TC-PAY-003) | High | Voucher booked as a negative line at 20% VAT: total VAT EUR 1.58 instead of EUR 1.75 | Fixed; BR-035 now states the pro rata allocation per rate group explicitly |
| DEF-058 | S5 accessibility check | High | Allergen line collapsed behind "Mehr" on small screens at 200% text | Fixed; FR-DSC-07 "never collapsed" added to the AC |
| DEF-066 | UAT-04 | Medium | No-show could be set before the start | Fixed; BR-030 clarified in SRS v1.2 (from start plus 15 minutes) |
| DEF-071 | UAT-08 | Medium | No tax category for small-business exempt businesses | Fixed; Exempt category in BR-015 (SRS v1.2) |
| DEF-089 | UAT-05 | High | New-order sound did not repeat on iPad Safari after the screen locked | Fixed; wake lock while the board is open |
| DEF-118 | R1.1 regression (TC-NFR-002) | Medium | Android cold start 3.4 s p90 against 3.0 s | Open; accepted for R1.1, fix in R1.2 |
| DEF-121 | R1.1 regression (TC-POS-007) | Low | Auto-pause email sent twice when two board tabs disconnect | Open; accepted for R1.1, fix in R1.2 |

DEF-047 deserves a note. Treating the voucher as a negative line at the standard rate understated VAT by EUR 0.17 on the canonical basket. On a reduced-rate-heavy menu it would have made every receipt disagree with the business's own VAT return. The worked example caught it because it was computed by hand before the code existed.

## 7. Schedule and status

| Cycle | Dates | Build | Result |
|---|---|---|---|
| R1 system test | Sprint by sprint, S1 to S7 | Per sprint | Exit met each sprint |
| R1 regression | 2026-06-08 to 2026-06-12 | 1.0.0-rc.3 | All P1 pass |
| UAT | 2026-06-15 to 2026-06-26 | 1.0.0-rc.4 | 10 of 10 scripts passed; signed 2026-06-26 |
| R1.1 regression | 2026-09-28 to 2026-10-02 | 1.1.0-rc.2 | See [test cases](test-cases.md): 2 failures with accepted defects, 1 case not run (restore drill on 2026-10-15) |

## 8. Roles

| Role | Responsibility |
|---|---|
| QA Lead | Strategy, plan, regression, release recommendation |
| QA engineer | Automation, integration and device tests |
| Business Analyst | Rule-based test design with QA; worked examples; UAT facilitation; requirement-gap classification |
| Developers | Unit and contract tests; fixing defects |
| Compliance Lead, Finance Lead | Review of allergen, privacy, VAT and payment test results |
| Pilot businesses and customer panel | UAT execution and sign-off |

## Related documents

- [Test cases](test-cases.md)
- [UAT plan and scripts](uat-plan-and-scripts.md)
- [Test data](test-data/README.md)
- [Requirements traceability matrix](../02-requirements/requirements-traceability-matrix.md)
- [Non-functional requirements](../02-requirements/non-functional-requirements.md)
- [Definition of ready and done](../05-delivery/definition-of-ready-and-done.md)
