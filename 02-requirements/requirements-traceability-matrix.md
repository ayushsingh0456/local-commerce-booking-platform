# Requirements Traceability Matrix: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | LC-REQ-RTM |
| Version | 1.3 (generated) |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-10-02 |
| Reviewers | Product Owner; QA Lead; Engineering Lead |

**Purpose and scope.** This matrix traces every requirement forward and backward:
- each business objective to the business need and epic;
- each functional requirement to its business rules, user stories, acceptance criteria, API operations and test cases, with its verification status;
- each business rule and non-functional requirement to its test cases.

The matrix is **generated** from the requirement model, the user-story files and the test-case catalog, so it cannot drift from them. The machine-readable version is [requirements-traceability-matrix.csv](requirements-traceability-matrix.csv). Verification status comes from the R1.1 regression cycle (2026-09-28 to 2026-10-02, build 1.1.0-rc.2).

## 1. Summary

| Measure | Value |
|---|---|
| Business objectives | 6 |
| Business needs and epics | 6 and 6 |
| Functional requirements | 76 |
| Functional requirements Verified | 75 of 76 |
| Functional requirements open | FR-POS-05 (Open: DEF-121) |
| Business rules with a test case | 45 of 45 |
| Non-functional requirements with a test case | 27 of 27 |
| User stories / acceptance criteria | 38 / 166 |
| API operations traced to FRs | 42 |
| Test cases | 98 |

## 2. Objectives to needs and epics

| Objective | Name | Epics | Business needs |
|---|---|---|---|
| OBJ-01 | Cut uncollected pickup orders | EP-01, EP-03 | BN-01, BN-03 |
| OBJ-02 | Cut appointment no-shows | EP-01, EP-04 | BN-01, BN-04 |
| OBJ-03 | Take bookings when the phone cannot | EP-02, EP-04 | BN-02, BN-04 |
| OBJ-04 | Complete and current allergen information | EP-02, EP-05 | BN-02, BN-05 |
| OBJ-05 | Onboard businesses quickly | EP-06 | BN-06 |
| OBJ-06 | Free counter staff at lunch | EP-03, EP-05 | BN-03, BN-05 |

## 3. Functional requirements

### ACC: Customer Account & Privacy (EP-01)

| FR | Priority | Release | Business rules | Stories (acceptance criteria) | API operations | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-ACC-01 | Must | R1 | BR-001 | US-001 (4 AC) | PUT /me/saved-locations/{locationId} | TC-ACC-001 | Verified |
| FR-ACC-02 | Must | R1 | BR-002, BR-003 | US-002 (5 AC) | POST /auth/sign-up | TC-ACC-002, TC-ACC-006 | Verified |
| FR-ACC-03 | Must | R1 | BR-004 | US-003 (5 AC) | POST /auth/sign-in | TC-ACC-004 | Verified |
| FR-ACC-04 | Must | R1 | BR-004 | US-004 (5 AC) | POST /auth/password-reset/codes<br>POST /auth/password-reset/verifications<br>POST /auth/password-reset/completions | TC-ACC-005 | Verified |
| FR-ACC-05 | Must | R1 | BR-002 | US-002 (5 AC) | POST /auth/sign-up | TC-ACC-003 | Verified |
| FR-ACC-06 | Should | R1 | BR-041 | US-005 (5 AC) | GET /me<br>PATCH /me | TC-ACC-009 | Verified |
| FR-ACC-07 | Must | R1 | BR-005 | US-005 (5 AC) | POST /me/deletion-requests | TC-ACC-007 | Verified |
| FR-ACC-08 | Must | R1 | BR-003 | US-002 (5 AC), US-005 (5 AC) | POST /auth/sign-up<br>PATCH /me | TC-ACC-006 | Verified |
| FR-ACC-09 | Should | R1 | BR-005 | US-005 (5 AC) | Platform Admin portal export (internal) | TC-ACC-008 | Verified |

### DSC: Discovery, Menus & Allergens (EP-02)

| FR | Priority | Release | Business rules | Stories (acceptance criteria) | API operations | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-DSC-01 | Must | R1 | BR-006, BR-007 | US-006 (4 AC) | GET /locations | TC-DSC-001 | Verified |
| FR-DSC-02 | Must | R1 | BR-009 | US-007 (4 AC) | GET /search | TC-DSC-007 | Verified |
| FR-DSC-03 | Must | R1 | BR-006, BR-007 | US-008 (4 AC) | GET /locations | TC-DSC-006 | Verified |
| FR-DSC-04 | Must | R1 | BR-007, BR-008 | US-006 (4 AC) | GET /locations | TC-DSC-002 | Verified |
| FR-DSC-05 | Must | R1 | BR-006 | US-009 (5 AC) | GET /locations/{locationId} | TC-DSC-005 | Verified |
| FR-DSC-06 | Must | R1 | BR-010, BR-014, BR-015 | US-009 (5 AC) | GET /locations/{locationId}/catalog | TC-DSC-003 | Verified |
| FR-DSC-07 | Must | R1 | BR-010, BR-011, BR-012 | US-010 (6 AC) | GET /locations/{locationId}/catalog | TC-DSC-004 | Verified |
| FR-DSC-08 | Must | R1 | BR-001 | US-001 (4 AC) | PUT /me/saved-locations/{locationId}<br>DELETE /me/saved-locations/{locationId} | TC-ACC-001, TC-DSC-009 | Verified |
| FR-DSC-09 | Should | R1 | BR-010, BR-012 | US-010 (6 AC) | GET /locations/{locationId}/catalog | TC-DSC-008 | Verified |

### ORD: Ordering & Pickup (EP-03)

| FR | Priority | Release | Business rules | Stories (acceptance criteria) | API operations | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-ORD-01 | Must | R1 | BR-016 | US-011 (4 AC) | Cart held on the device; sent with quoteCheckout and placeOrder | TC-ORD-001 | Verified |
| FR-ORD-02 | Must | R1 | BR-017, BR-018 | US-012 (5 AC) | GET /locations/{locationId}/pickup-slots | TC-ORD-002, TC-ORD-003 | Verified |
| FR-ORD-03 | Must | R1 | BR-019 | US-012 (5 AC) | POST /slot-holds | TC-ORD-005 | Verified |
| FR-ORD-04 | Must | R1 | BR-035, BR-036 | US-014 (5 AC) | POST /checkout-quotes | TC-ORD-004, TC-PAY-003 | Verified |
| FR-ORD-05 | Must | R1 | BR-021, BR-024 | US-015 (5 AC) | POST /orders | TC-ORD-006 | Verified |
| FR-ORD-06 | Must | R1 | BR-021, BR-041 | US-015 (5 AC) | GET /me/bookings/{bookingId} | TC-ORD-013 | Verified |
| FR-ORD-07 | Must | R1 | BR-022 | US-015 (5 AC) | System timeout job (no client call) | TC-ORD-007 | Verified |
| FR-ORD-08 | Must | R1 | BR-023 | US-016 (4 AC) | POST /me/bookings/{bookingId}/cancellation | TC-ORD-008 | Verified |
| FR-ORD-09 | Must | R1 | BR-021 | US-015 (5 AC) | GET /me/bookings<br>GET /me/bookings/{bookingId} | TC-ORD-013 | Verified |
| FR-ORD-10 | Must | R1 | BR-025 | US-017 (4 AC) | POST /orders/{orderId}/rebook | TC-ORD-009 | Verified |
| FR-ORD-11 | Should | R1 | BR-017 | US-018 (3 AC) | GET /locations/{locationId}/pickup-slots | TC-ORD-012 | Verified |
| FR-ORD-12 | Must | R1.1 | BR-013, BR-019, BR-020 | US-020 (5 AC) | POST /orders | TC-ORD-010, TC-ORD-011 | Verified |

### PAY: Payments & Receipts (EP-03)

| FR | Priority | Release | Business rules | Stories (acceptance criteria) | API operations | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-PAY-01 | Must | R1 | BR-032 | US-013 (6 AC) | POST /orders | TC-PAY-001 | Verified |
| FR-PAY-02 | Must | R1 | BR-033 | US-013 (6 AC) | POST /webhooks/payment-provider | TC-PAY-002 | Verified |
| FR-PAY-03 | Should | R1 | BR-032, BR-024 | US-013 (6 AC) | POST /orders<br>POST /business/orders/{orderId}/transitions | TC-PAY-009, TC-POS-004 | Verified |
| FR-PAY-04 | Must | R1 | BR-034, BR-023 | US-016 (4 AC), US-032 (4 AC) | POST /business/orders/{orderId}/refunds | TC-PAY-005 | Verified |
| FR-PAY-05 | Must | R1 | BR-035, BR-015 | US-014 (5 AC) | POST /checkout-quotes | TC-PAY-003, TC-PAY-010 | Verified |
| FR-PAY-06 | Must | R1 | BR-036 | US-014 (5 AC) | POST /checkout-quotes | TC-PAY-004 | Verified |
| FR-PAY-07 | Must | R1 | BR-038 | US-019 (3 AC) | POST /orders<br>GET /me/bookings/{bookingId} | TC-PAY-006, TC-PAY-010 | Verified |
| FR-PAY-08 | Must | R1 | BR-037 | US-032 (4 AC) | GET /business/statements | TC-PAY-008 | Verified |
| FR-PAY-09 | Must | R1 | BR-033 | US-013 (6 AC) | POST /webhooks/payment-provider | TC-PAY-007 | Verified |

### APT: Appointment Booking (EP-04)

| FR | Priority | Release | Business rules | Stories (acceptance criteria) | API operations | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-APT-01 | Must | R1 | BR-015 | US-021 (3 AC) | GET /locations/{locationId}/catalog | TC-APT-001 | Verified |
| FR-APT-02 | Must | R1 | BR-026, BR-027 | US-021 (3 AC), US-022 (5 AC) | GET /locations/{locationId}/availability | TC-APT-001, TC-APT-002, TC-APT-003 | Verified |
| FR-APT-03 | Must | R1 | BR-028 | US-023 (5 AC) | POST /slot-holds | TC-APT-004 | Verified |
| FR-APT-04 | Must | R1 | BR-027, BR-028 | US-023 (5 AC) | POST /appointments | TC-APT-004, TC-APT-005, TC-APT-006 | Verified |
| FR-APT-05 | Must | R1 | BR-041 | US-024 (5 AC) | Reminder job (no client call) | TC-APT-007 | Verified |
| FR-APT-06 | Must | R1 | BR-029 | US-024 (5 AC) | POST /me/bookings/{bookingId}/cancellation<br>POST /appointments | TC-APT-008 | Verified |
| FR-APT-07 | Must | R1 | BR-030 | US-025 (4 AC) | GET /me/bookings/{bookingId}<br>POST /business/appointments/{appointmentId}/transitions | TC-APT-009 | Verified |
| FR-APT-08 | Should | R1 | n/a | US-023 (5 AC) | POST /appointments<br>GET /business/locations/{locationId}/appointments | TC-APT-004 | Verified |
| FR-APT-09 | Must | R1 | BR-032 | US-023 (5 AC) | POST /appointments | TC-APT-004 | Verified |
| FR-APT-10 | Must | R1 | BR-031 | US-025 (4 AC) | POST /business/appointments/{appointmentId}/transitions | TC-APT-010 | Verified |

### CAT: Catalog & Business Setup (EP-05)

| FR | Priority | Release | Business rules | Stories (acceptance criteria) | API operations | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-CAT-01 | Must | R1 | BR-043 | US-036 (4 AC) | Configuration endpoints (API guidelines section 9) | TC-CAT-010 | Verified |
| FR-CAT-02 | Must | R1 | BR-007, BR-017, BR-018, BR-023 | US-036 (4 AC) | PUT /business/locations/{locationId} | TC-CAT-005 | Verified |
| FR-CAT-03 | Must | R1 | BR-014, BR-015 | US-030 (4 AC) | PUT /business/items/{itemId} | TC-CAT-001 | Verified |
| FR-CAT-04 | Must | R1 | BR-010, BR-011, BR-013 | US-029 (5 AC) | POST /business/items/{itemId}/allergen-declarations | TC-CAT-002, TC-CAT-003 | Verified |
| FR-CAT-05 | Must | R1 | BR-012 | US-029 (5 AC) | PUT /business/items/{itemId} | TC-CAT-002, TC-CAT-004 | Verified |
| FR-CAT-06 | Must | R1 | BR-026 | US-031 (4 AC) | PUT /business/services/{serviceId} | TC-CAT-006 | Verified |
| FR-CAT-07 | Must | R1 | BR-039 | US-031 (4 AC) | Configuration endpoints (API guidelines section 9) | TC-CAT-008 | Verified |
| FR-CAT-08 | Must | R1 | BR-031 | US-031 (4 AC) | PUT /business/staff/{staffId}/working-hours | TC-CAT-007 | Verified |
| FR-CAT-09 | Should | R1 | BR-036 | US-030 (4 AC) | Configuration endpoints (API guidelines section 9) | TC-CAT-009 | Verified |

### POS: Order Board & Day Operations (POS view) (EP-05)

| FR | Priority | Release | Business rules | Stories (acceptance criteria) | API operations | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-POS-01 | Must | R1 | BR-040, BR-022 | US-026 (5 AC) | GET /business/locations/{locationId}/orders | TC-POS-001 | Verified |
| FR-POS-02 | Must | R1 | BR-022, BR-033 | US-026 (5 AC) | POST /business/orders/{orderId}/transitions | TC-POS-002 | Verified |
| FR-POS-03 | Must | R1 | BR-024, BR-021 | US-027 (4 AC) | POST /business/orders/{orderId}/transitions | TC-POS-003, TC-POS-004 | Verified |
| FR-POS-04 | Must | R1 | BR-014 | US-028 (4 AC) | PUT /business/locations/{locationId}/item-availability/{itemId} | TC-POS-005 | Verified |
| FR-POS-05 | Must | R1 | BR-040 | US-028 (4 AC) | PUT /business/locations/{locationId}/ordering-pause | TC-POS-007 | Open: DEF-121 |
| FR-POS-06 | Must | R1 | BR-030, BR-031 | US-025 (4 AC) | GET /business/locations/{locationId}/appointments<br>POST /business/appointments/{appointmentId}/transitions | TC-APT-009, TC-POS-010 | Verified |
| FR-POS-07 | Must | R1 | BR-039 | US-026 (5 AC) | Enforced on every Business operation (403 for Staff) | TC-POS-008 | Verified |
| FR-POS-08 | Should | R1 | BR-024 | US-027 (4 AC) | POST /business/orders/{orderId}/transitions | TC-POS-009 | Verified |
| FR-POS-09 | Must | R1 | BR-012 | US-026 (5 AC) | GET /business/locations/{locationId}/orders | TC-POS-006 | Verified |

### ONB: Business Onboarding & Platform Administration (EP-06)

| FR | Priority | Release | Business rules | Stories (acceptance criteria) | API operations | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-ONB-01 | Must | R1 | BR-042 | US-033 (4 AC) | POST /business-applications | TC-ONB-001 | Verified |
| FR-ONB-02 | Must | R1 | BR-042, BR-043 | US-034 (4 AC) | GET /platform/business-applications<br>POST /platform/business-applications/{applicationId}/decisions | TC-ONB-002 | Verified |
| FR-ONB-03 | Must | R1 | BR-043 | US-035 (3 AC) | POST /webhooks/payment-provider | TC-ONB-003 | Verified |
| FR-ONB-04 | Must | R1 | BR-043 | US-036 (4 AC) | POST /platform/businesses/{businessId}/status-changes | TC-ONB-004 | Verified |
| FR-ONB-05 | Must | R1 | BR-044 | US-037 (4 AC) | POST /platform/businesses/{businessId}/status-changes | TC-ONB-005 | Verified |
| FR-ONB-06 | Must | R1 | BR-015, BR-035 | US-038 (4 AC) | Configuration endpoints (API guidelines section 9) | TC-ONB-006 | Verified |
| FR-ONB-07 | Must | R1 | BR-045 | US-038 (4 AC) | Configuration endpoints (API guidelines section 9) | TC-ONB-007 | Verified |
| FR-ONB-08 | Should | R1 | BR-008 | US-037 (4 AC) | Configuration endpoints (API guidelines section 9) | TC-ONB-008 | Verified |
| FR-ONB-09 | Should | R1 | BR-036 | US-038 (4 AC) | Configuration endpoints (API guidelines section 9) | TC-ONB-009 | Verified |

## 4. Business rules to test cases

| Rule | Related FRs | Stories | Test cases | Automated |
|---|---|---|---|---|
| BR-001 | FR-ACC-01, FR-DSC-08 | US-001 | TC-ACC-001, TC-DSC-009 | Yes |
| BR-002 | FR-ACC-02, FR-ACC-05 | US-002 | TC-ACC-002, TC-ACC-003 | Yes |
| BR-003 | FR-ACC-02, FR-ACC-08 | US-002, US-005 | TC-ACC-006 | Yes |
| BR-004 | FR-ACC-03, FR-ACC-04 | US-003, US-004 | TC-ACC-004, TC-ACC-005 | Yes |
| BR-005 | FR-ACC-07, FR-ACC-09 | US-005 | TC-ACC-007, TC-ACC-008 | Yes |
| BR-006 | FR-DSC-01, FR-DSC-03, FR-DSC-05 | US-006, US-008, US-009 | TC-DSC-001, TC-DSC-005, TC-DSC-006 | Yes |
| BR-007 | FR-DSC-01, FR-DSC-03, FR-DSC-04, FR-CAT-02 | US-006, US-008, US-036 | TC-DSC-002, TC-DSC-006, TC-CAT-005 | Yes |
| BR-008 | FR-DSC-04, FR-ONB-08 | US-006, US-037 | TC-DSC-002, TC-ONB-008 | Yes |
| BR-009 | FR-DSC-02 | US-007 | TC-DSC-007 | Yes |
| BR-010 | FR-DSC-06, FR-DSC-07, FR-DSC-09, FR-CAT-04 | US-009, US-010, US-029 | TC-DSC-003, TC-DSC-004, TC-DSC-008 | Yes |
| BR-011 | FR-DSC-07, FR-CAT-04 | US-010, US-029 | TC-DSC-004, TC-CAT-002 | Yes |
| BR-012 | FR-DSC-07, FR-DSC-09, FR-CAT-05, FR-POS-09 | US-010, US-026, US-029 | TC-DSC-004, TC-DSC-008, TC-CAT-002, TC-CAT-004, TC-POS-006 | Yes |
| BR-013 | FR-ORD-12, FR-CAT-04 | US-020, US-029 | TC-ORD-011, TC-CAT-003 | Yes |
| BR-014 | FR-DSC-06, FR-CAT-03, FR-POS-04 | US-009, US-028, US-030 | TC-DSC-003, TC-POS-005 | Yes |
| BR-015 | FR-DSC-06, FR-PAY-05, FR-APT-01, FR-CAT-03, FR-ONB-06 | US-009, US-014, US-021, US-030, US-038 | TC-DSC-003, TC-PAY-010, TC-APT-001, TC-CAT-001, TC-ONB-006 | Yes |
| BR-016 | FR-ORD-01 | US-011 | TC-ORD-001 | Yes |
| BR-017 | FR-ORD-02, FR-ORD-11, FR-CAT-02 | US-012, US-018, US-036 | TC-ORD-002, TC-ORD-012, TC-CAT-005 | Yes |
| BR-018 | FR-ORD-02, FR-CAT-02 | US-012, US-036 | TC-ORD-003, TC-CAT-005 | Yes |
| BR-019 | FR-ORD-03, FR-ORD-12 | US-012, US-020 | TC-ORD-005, TC-ORD-010 | Yes |
| BR-020 | FR-ORD-12 | US-020 | TC-ORD-010 | Yes |
| BR-021 | FR-ORD-05, FR-ORD-06, FR-ORD-09, FR-POS-03 | US-015, US-027 | TC-ORD-006, TC-ORD-013, TC-POS-003 | Yes |
| BR-022 | FR-ORD-07, FR-POS-01, FR-POS-02 | US-015, US-026 | TC-ORD-007, TC-POS-001, TC-POS-002 | Yes |
| BR-023 | FR-ORD-08, FR-PAY-04, FR-CAT-02 | US-016, US-032, US-036 | TC-ORD-008, TC-CAT-005 | Yes |
| BR-024 | FR-ORD-05, FR-PAY-03, FR-POS-03, FR-POS-08 | US-013, US-015, US-027 | TC-PAY-009, TC-POS-003, TC-POS-004, TC-POS-009 | Yes |
| BR-025 | FR-ORD-10 | US-017 | TC-ORD-009 | Yes |
| BR-026 | FR-APT-02, FR-CAT-06 | US-021, US-022, US-031 | TC-APT-002, TC-APT-003, TC-CAT-006 | Yes |
| BR-027 | FR-APT-02, FR-APT-04 | US-021, US-023 | TC-APT-001, TC-APT-006 | Yes |
| BR-028 | FR-APT-03, FR-APT-04 | US-023 | TC-APT-004, TC-APT-005 | Yes |
| BR-029 | FR-APT-06 | US-024 | TC-APT-008 | Yes |
| BR-030 | FR-APT-07, FR-POS-06 | US-025 | TC-APT-009, TC-POS-010 | Yes |
| BR-031 | FR-APT-10, FR-CAT-08, FR-POS-06 | US-025, US-031 | TC-APT-010, TC-CAT-007 | Yes |
| BR-032 | FR-PAY-01, FR-PAY-03, FR-APT-09 | US-013, US-023 | TC-PAY-001, TC-PAY-009, TC-APT-004 | Yes |
| BR-033 | FR-PAY-02, FR-PAY-09, FR-POS-02 | US-013, US-026 | TC-PAY-002, TC-PAY-007, TC-POS-002 | Yes |
| BR-034 | FR-PAY-04 | US-016, US-032 | TC-PAY-005 | Yes |
| BR-035 | FR-ORD-04, FR-PAY-05, FR-ONB-06 | US-014, US-038 | TC-ORD-004, TC-PAY-003, TC-ONB-006 | Yes |
| BR-036 | FR-ORD-04, FR-PAY-06, FR-CAT-09, FR-ONB-09 | US-014, US-030, US-038 | TC-ORD-004, TC-PAY-004, TC-CAT-009, TC-ONB-009 | Yes |
| BR-037 | FR-PAY-08 | US-032 | TC-PAY-008, TC-ONB-009 | Yes |
| BR-038 | FR-PAY-07 | US-019 | TC-PAY-006 | Yes |
| BR-039 | FR-CAT-07, FR-POS-07 | US-026, US-031 | TC-CAT-008, TC-POS-008 | Yes |
| BR-040 | FR-POS-01, FR-POS-05 | US-026, US-028 | TC-POS-001, TC-POS-007 | Yes |
| BR-041 | FR-ACC-06, FR-ORD-06, FR-APT-05 | US-005, US-015, US-024 | TC-ACC-009, TC-ORD-013, TC-APT-007 | Yes |
| BR-042 | FR-ONB-01, FR-ONB-02 | US-033, US-034 | TC-ONB-001, TC-ONB-002 | Yes |
| BR-043 | FR-CAT-01, FR-ONB-02, FR-ONB-03, FR-ONB-04 | US-034, US-035, US-036 | TC-CAT-010, TC-ONB-002, TC-ONB-003, TC-ONB-004 | Yes |
| BR-044 | FR-ONB-05 | US-037 | TC-ONB-005 | Yes |
| BR-045 | FR-ONB-07 | US-038 | TC-CAT-003, TC-ONB-007 | Yes |

## 5. Non-functional requirements to test cases

| NFR | Characteristic | Test cases | Status |
|---|---|---|---|
| NFR-PERF-01 | Performance efficiency - time behavior | TC-NFR-001 | Verified |
| NFR-PERF-02 | Performance efficiency - time behavior | TC-NFR-002 | Open: DEF-118 |
| NFR-PERF-03 | Performance efficiency - time behavior | TC-NFR-003 | Verified |
| NFR-PERF-04 | Performance efficiency - time behavior | TC-NFR-004 | Verified |
| NFR-CAP-01 | Performance efficiency - capacity | TC-NFR-001 | Verified |
| NFR-AVL-01 | Reliability - availability | TC-NFR-014 | Partly verified (drill scheduled 2026-10-15) |
| NFR-AVL-02 | Reliability - recoverability | TC-NFR-014 | Partly verified (drill scheduled 2026-10-15) |
| NFR-AVL-03 | Reliability - fault tolerance | TC-NFR-007 | Verified |
| NFR-SEC-01 | Security - integrity | TC-NFR-005 | Verified |
| NFR-SEC-02 | Security - confidentiality | TC-NFR-015 | Verified |
| NFR-SEC-03 | Security - authenticity | TC-NFR-005 | Verified |
| NFR-SEC-04 | Security - authenticity | TC-NFR-005 | Verified |
| NFR-SEC-05 | Security - accountability | TC-NFR-006 | Verified |
| NFR-PRIV-01 | Security - confidentiality (privacy) | TC-NFR-009, TC-NFR-018 | Verified |
| NFR-PRIV-02 | Security - confidentiality (privacy) | TC-NFR-018 | Verified |
| NFR-USE-01 | Usability - operability | TC-NFR-008 | Verified |
| NFR-USE-02 | Usability - user error protection | TC-NFR-010 | Verified |
| NFR-ACC-01 | Usability - accessibility | TC-NFR-008 | Verified |
| NFR-I18N-01 | Usability - learnability (localization) | TC-NFR-010 | Verified |
| NFR-CMP-01 | Compatibility - co-existence | TC-NFR-016 | Verified |
| NFR-MOB-01 | Performance efficiency - resource utilization | TC-NFR-002 | Open: DEF-118 |
| NFR-DAT-01 | Functional suitability - correctness | TC-NFR-011 | Verified |
| NFR-DAT-02 | Functional suitability - correctness | TC-NFR-012 | Verified |
| NFR-MNT-01 | Maintainability - testability | TC-NFR-017 | Verified |
| NFR-MNT-02 | Maintainability - modifiability | TC-NFR-017 | Verified |
| NFR-OBS-01 | Maintainability - analyzability | TC-NFR-009 | Verified |
| NFR-OBS-02 | Reliability - maturity | TC-NFR-013 | Verified |

## 6. How the matrix is maintained

- A change request that changes a requirement updates the model, the SRS and the affected stories in the same pull request; the matrix is regenerated by CI and the diff is reviewed with the CR.
- The generator stops if an FR, business rule, NFR or story has no test case, if a test case cites an unknown ID, or if a placeholder cannot be resolved.
- Open items are listed in the summary with their defect IDs; they are tracked in the [test strategy](../06-quality/test-strategy-and-plan.md) defect summary.

## Related documents

- [Business Requirements Document](BRD.md)
- [Software Requirements Specification](SRS.md)
- [Business rules](business-rules.md)
- [Non-functional requirements](non-functional-requirements.md)
- [Epics](../05-delivery/epics.md)
- [Test cases](../06-quality/test-cases.md)
- [OpenAPI specification](../04-api/openapi.yaml)
