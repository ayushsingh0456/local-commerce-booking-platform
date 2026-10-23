# Test Cases: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | QT-QA-02 |
| Version | 1.4 |
| Status | Baselined |
| Owner | Business Analyst (co-authored with the QA Lead) |
| Last updated | 2026-10-02 |
| Reviewers | QA Lead, Engineering Lead, Product Owner, Compliance Lead, Finance Lead |

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-05-29 | R1 system test cases |
| 1.1 | 2026-06-12 | Boundary rows from DEF-047, DEF-058, DEF-066 and DEF-071; SRS v1.2 |
| 1.2 | 2026-06-26 | Last Result from the R1 regression and UAT |
| 1.3 | 2026-09-15 | TC-ORD-011 (CR-005, INC-2026-006); TC-NFR-012 freshness probe |
| 1.4 | 2026-10-02 | TC-ORD-010 (CR-006, INC-2026-011); TC-NFR-013 anomaly alerts; Last Result from the R1.1 regression |

## Purpose and scope

[test-cases.csv](test-cases.csv) is the test case specification for the platform R1 and R1.1: 98 cases that cover every functional requirement, business rule, user story and non-functional requirement. This guide explains how to read the file, summarizes coverage, and writes out the cases where the arithmetic or the timing is the point. A reviewer should be able to check those expected results by hand.

## How to read the CSV

| Column | Meaning |
|---|---|
| TC ID | `TC-<MOD>-NNN`; MOD is the SRS module (ACC, DSC, ORD, PAY, APT, CAT, POS, ONB) or NFR |
| Title | What the case proves; "(INC-... regression, CR-...)" marks a case added after an incident |
| Module, Type | Type is one of Functional, Negative, Boundary, Integration, Security, Performance, Reliability, Accessibility, Usability, Compatibility, Maintainability |
| Technique | Design technique ([test strategy](test-strategy-and-plan.md#3-test-design-techniques)) |
| Priority | P1 runs in every release and blocks it on failure; P2 runs in every release; P3 in full regression only |
| Preconditions | State before step 1, including the virtual-clock time where it matters |
| Test Data | `file: row IDs` from [test-data](test-data/README.md), or location and order IDs |
| Steps, Expected Result | Numbered; expected results match the steps |
| Requirement IDs, Business Rule IDs, User Story IDs | Traceability; all IDs exist in the SRS, the business rules and the stories |
| Automation | Automated, Manual or Candidate |
| Last Result | R1.1 regression, 2026-09-28 to 2026-10-02, build 1.1.0-rc.2, with the defect ID for failures |

Conventions: times are Europe/Vienna; amounts are shown as in the UI ("€ 15,70") or in cents where the API is checked; the virtual clock is set per test location in staging only.

## Coverage summary

| Module | Cases | Functional | Negative | Boundary | Integration | Security | Performance | Reliability | Accessibility | Usability | Compatibility | Maintainability | P1 | Automated | Requirements covered |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ACC | 9 | 5 | 1 | 3 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 7 | 8 | 9 of 9 FRs |
| DSC | 9 | 7 | 0 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 5 | 9 | 9 of 9 FRs |
| ORD | 13 | 5 | 0 | 7 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 10 | 13 | 12 of 12 FRs |
| PAY | 10 | 4 | 0 | 3 | 2 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 8 | 10 | 9 of 9 FRs |
| APT | 10 | 3 | 1 | 5 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 7 | 10 | 10 of 10 FRs |
| CAT | 10 | 4 | 1 | 3 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 4 | 9 | 9 of 9 FRs |
| POS | 10 | 7 | 1 | 1 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 6 | 10 | 9 of 9 FRs |
| ONB | 9 | 4 | 0 | 2 | 1 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 6 | 9 | 9 of 9 FRs |
| NFR | 18 | 2 | 0 | 0 | 0 | 5 | 4 | 3 | 1 | 1 | 1 | 1 | 9 | 13 | 27 of 27 NFRs |
| **Total** | **98** | **41** | **4** | **26** | **5** | **10** | **4** | **3** | **2** | **1** | **1** | **1** | **62** | **91** |  |

| Coverage measure | Result |
|---|---|
| Functional requirements with at least one case | 76 of 76 |
| Business rules with at least one case | 45 of 45 |
| Business rules with at least one automated case (NFR-MNT-01) | 45 of 45 |
| User stories linked to at least one case | 38 of 38 |
| Non-functional requirements covered by a case | 27 of 27 |
| Last Result (R1.1 regression, 2026-09-28 to 2026-10-02, build 1.1.0-rc.2) | Pass 95, Fail 2, Not run 1 |

The generator that writes the CSV stops if any FR, business rule, NFR or story lacks a case, or if a case cites an unknown ID.

## Featured test cases

### 1. TC-PAY-003: the canonical VAT basket with voucher MOHN3

| Field | Value |
|---|---|
| Requirements | FR-PAY-05, FR-ORD-04 |
| Business rule | BR-035 (discount allocated pro rata by rate group, highest rate takes the remainder; VAT per group, rounded half up) |
| Story | US-014 (AC1, AC2) |
| Data | menu-items.csv IT-1001 (2 x), IT-1002, IT-1003 with CH-1003-2; voucher MOHN3 |
| Technique | Decision table, expected values computed by hand before implementation |

| Step | 10% group | 20% group | Total |
|---|---|---|---|
| Gross before discount | 680 + 690 = 1370 | 390 + 110 = 500 | 1870 |
| Discount share | 300 x 1370 / 1870 = 219.79, so 220 | 300 - 220 = 80 | 300 |
| Gross after discount | 1150 | 420 | 1570 |
| VAT | 1150 x 10 / 110 = 104.55, so 105 | 420 x 20 / 120 = 70 | 175 |
| Net | 1045 | 350 | 1395 |

Also checked, as the DEF-047 regression: treating the voucher as a negative line at 20% would give 125 + 83 - 50 = 158; the system must return 175. The receipt, the quote and `order_vat_groups` carry identical values. Commission on 1570 is 126 plus VAT 25; payout 1419 (TC-PAY-008).

### 2. TC-APT-002 and TC-APT-003: salon start times with a buffer

| Field | Value |
|---|---|
| Requirement | FR-APT-02 |
| Business rule | BR-026 (grid, notice, horizon, [s, s + duration + buffer) inside one working interval, no overlap, half-open intervals) |
| Story | US-022 (AC1 to AC5) |
| Data | salon-schedule.csv SS-01 (Lena 09:00 to 13:00), SS-03 (appointment occupying 10:30 to 11:15), SS-05 (Damenhaarschnitt 45 + 15) |

| Start | Occupies | Expected | Why |
|---|---|---|---|
| 09:00, 09:15 | to 10:00, 10:15 | Offered | Fits |
| 09:30 | to 10:30 | Offered | Touches the booking at 10:30 |
| 09:45 to 11:00 | overlaps | Not offered | Overlap with 10:30 to 11:15 |
| 11:15 | to 12:15 | Offered | Starts when the buffer ends |
| 11:30, 11:45 | to 12:30, 12:45 | Offered | Fits |
| 12:00 | to 13:00 | Offered | Buffer ends at the end of the shift |
| 12:15 | to 13:15 | Not offered | Buffer exceeds the shift |

Expected list: 09:00, 09:15, 09:30, 11:15, 11:30, 11:45, 12:00 (7). After booking 09:30: 11:15, 11:30, 11:45, 12:00 (4). Notice: at 07:15 the first time is 09:15; at 07:16 it is 09:30. Horizon from 2026-10-12: 2026-12-07 bookable, 2026-12-08 not.

### 3. TC-ORD-010: slot filled during a slow bank challenge (INC-2026-011 regression)

| Time | Event | Expected |
|---|---|---|
| 11:40:00 | Sophie holds 12:15 (5 of 6 used by orders) | Hold expires 11:50:00 |
| 11:43:10 | Challenge starts | Hold extended once to 11:55:00 |
| 11:55:00 | Hold expires | Unit free |
| 11:55:30 | Customer B orders 12:15 | 6 of 6 |
| 11:56:20 | Sophie's bank approves | 409 SLOT_FULL_AT_AUTHORISATION; 12:30, 12:45, 13:00 held 3 minutes; no order in 12:15 |
| 11:57:40 | Sophie chooses 12:30 | Order Placed in 12:30 with the same authorization |

The case uses the sandbox's configurable slow-challenge card. Before CR-006, step 5 created a seventh order in 12:15. That is exactly how 23 orders went over capacity in the incident.

### 4. TC-ORD-011: allergens changed after the item was added (INC-2026-006 regression)

The cart line stores declaration version 3 (A C G) at 08:40. The bakery confirms version 4 (A C G H) at 08:52. At 09:05 the order button must not start payment; the line shows "Neu: H Schalenfrüchte" in bold, and payment is possible only after the customer ticks "Neue Allergenangaben gelesen". The order line stores version 4. Row V3 (allergens removed only) still requires confirmation.

### 5. TC-ORD-002 and TC-ORD-008: the clock decides

| Case | Clock | Expected |
|---|---|---|
| Earliest slot, lead time 20 min | 11:52 / 11:55 / 11:56 | 12:15 / 12:15 / 12:30 |
| Cancel Accepted order, slot 12:15, cutoff 30 min | 11:45 / 11:46 | Canceled with refund / call the business |
| Acceptance timeout, placed 11:53:30 | Accept 11:58:29 / no action until 11:58:30 | Accepted / auto-rejected and voided (TC-ORD-007) |
| Not collected, slot 12:15 to 12:30 | 13:29 / 13:30 | Refused / allowed (TC-POS-009) |

### 6. TC-PAY-005: refunds and the captured-amount cap

On the canonical order (1570 captured), a refund of 500 is allocated 366 to 10% (500 x 1150 / 1570 = 366.24) and 134 to 20%. VAT reversed is 33 and 22, and commission reversed is 40 plus 8 VAT. The remaining refundable amount is then 1070: a refund of 1071 is refused and 1070 is accepted.

### 7. TC-DSC-008: the allergen filter never leaves the phone

With the filter set to H, Nussschnecke is dimmed "Contains H" (row M1), Linsen-Bowl shows "Depending on choice: H" (M2), and Topfengolatsche has no label (M3). A network proxy and the error-monitoring sandbox capture every request during browsing and ordering; none contains the filter. This is a P1 case because the design depends on it (GDPR Article 9, DEC-10).

## Related documents

- [Test strategy and plan](test-strategy-and-plan.md)
- [UAT plan and scripts](uat-plan-and-scripts.md)
- [Test data](test-data/README.md)
- [Requirements traceability matrix](../02-requirements/requirements-traceability-matrix.md)
- [Business rules](../02-requirements/business-rules.md)
