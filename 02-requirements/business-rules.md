# Business Rules Catalog and Decision Tables: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | LC-REQ-BR |
| Version | 1.3 (aligned with SRS v1.3) |
| Status | Approved (baselined) |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Product Owner; Engineering Lead; QA Lead; Compliance Lead; Finance Lead; Partner Operations Lead |

**Purpose and scope.** This document holds the full catalog of the 45 business rules that constrain the platform's behavior, independent of any screen or API. It also gives decision tables for the rules whose combinations are easy to get wrong. The rule wording is canonical. The decision tables are the specification that developers implement and QA tests against. Every business rule needs at least one automated test (NFR-MNT-01).

## 1. Conventions

- **Rule wording** is canonical and identical in the [SRS](SRS.md), the user stories and the [traceability matrix](requirements-traceability-matrix.md).
- **Related FRs** are the functional requirements that implement a rule.
- **User stories** are the stories whose acceptance criteria exercise it.
- **Numbers are defaults.** Where a rule gives a range, the business or the platform can change the value; who can change what is in [SRS section 2.7](SRS.md#27-configuration-defaults).
- **Enforcement.** Rules are enforced server-side. The apps may pre-check for a better experience, but they are never the control.
- **Hit policy** of each decision table, using the DMN terms:
  - **Unique:** exactly one row matches.
  - **First:** rows are evaluated top-down and the first match wins.
  - **Collect:** every matching row applies.
- **Time.** Every time is Europe/Vienna local time. Intervals are half-open, [start, end): an interval that ends at 10:30 does not overlap one that starts at 10:30.
- **Money.** Amounts are EUR including VAT unless a column says net. "Rounded" always means half up to the cent.

Rules by area:

| Area | Rules | Count |
|---|---|---|
| Accounts and privacy | BR-001 to BR-005 | 5 |
| Discovery | BR-006 to BR-009 | 4 |
| Catalog and allergens | BR-010 to BR-015 | 6 |
| Ordering and pickup | BR-016 to BR-025 | 10 |
| Appointments | BR-026 to BR-031 | 6 |
| Payments | BR-032 to BR-038 | 7 |
| Business portal | BR-039 to BR-041 | 3 |
| Onboarding and platform | BR-042 to BR-045 | 4 |
| **Total** |  | **45** |

## 2. Business rule catalog

### Accounts and privacy (BR-001 to BR-005)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-001 | A guest can browse Home, Search, Category Listing, Business Detail, the full menu or service list with prices and allergens, and Item Details. Adding to a cart, ordering, booking, saving a business and opening Bookings need an account; after sign-in the customer returns to the action that was blocked. | FR-ACC-01, FR-DSC-08 | US-001 |
| BR-002 | Sign-up asks only for first and last name, email, mobile number, a password and the self-declaration "I am at least 14 years old" (no date of birth, no address). The declaration is not verified. An email address or mobile number belongs to at most one customer account, and an email address used for a portal account cannot be used for a customer account. | FR-ACC-02, FR-ACC-05 | US-002 |
| BR-003 | Marketing email, marketing push and analytics are separate opt-ins that start unticked and do not depend on accepting the terms. Each consent is stored with its version, source and timestamp and can be withdrawn at any time with the same ease. | FR-ACC-02, FR-ACC-08 | US-002, US-005 |
| BR-004 | Passwords have 8 or more characters, mix upper- and lower-case letters, contain at least one digit and must not appear in a breached-password list. Five consecutive failed password sign-ins lock password sign-in for 15 minutes. A reset code has 6 digits, is valid for 10 minutes, is single-use, allows 5 entry attempts, can be resent after 60 seconds (which invalidates the earlier code); the new password must differ from the current one and saving it ends all other sessions. | FR-ACC-03, FR-ACC-04 | US-003, US-004 |
| BR-005 | Account deletion is refused while an order is open or an appointment is upcoming. Otherwise personal data is erased or pseudonymized within 30 days; order, payment and receipt records are kept in pseudonymized form for 7 years to meet Austrian bookkeeping retention (BAO section 132). | FR-ACC-07, FR-ACC-09 | US-005 |

### Discovery (BR-006 to BR-009)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-006 | A business appears in lists and search only when it is Live and has an active location. A business with several locations appears once per location. Lists are sorted by straight-line distance ascending (km to one decimal, decimal comma); without location permission they are sorted by name A to Z and the list header states this. | FR-DSC-01, FR-DSC-03, FR-DSC-05 | US-006, US-008, US-009 |
| BR-007 | Open state is computed per location in Europe/Vienna time from weekly hours and dated closures, and is always shown in words: "Geöffnet bis HH:mm", "Öffnet um HH:mm" or "Heute geschlossen". | FR-DSC-01, FR-DSC-03, FR-DSC-04, FR-CAT-02 | US-006, US-008, US-036 |
| BR-008 | Lists carry no star ratings, review totals, sponsored markers or paid placement; ranking uses only distance or name, and these parameters are disclosed in the business terms. Customer feedback is private to the platform and never shown publicly. | FR-DSC-04, FR-ONB-08 | US-006, US-037 |
| BR-009 | Search runs from 2 characters, is case- and accent-insensitive ("kaese" finds "Käse"), covers business names and types, item names and service names, and returns at most 20 results per group (Businesses, Dishes, Services). | FR-DSC-02 | US-007 |

### Catalog and allergens (BR-010 to BR-015)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-010 | Allergens are the 14 of Regulation (EU) No 1169/2011 Annex II, shown with the Austrian letter codes: A gluten-containing cereals, B crustaceans, C eggs, D fish, E peanuts, F soybeans, G milk (including lactose), H tree nuts, L celery, M mustard, N sesame, O sulphur dioxide and sulphites, P lupin, R molluscs. | FR-DSC-06, FR-DSC-07, FR-DSC-09, FR-CAT-04 | US-009, US-010, US-029 |
| BR-011 | A food item or option choice can be published for ordering only with a complete allergen declaration: one or more allergens, or an explicit confirmation by a Business Admin that it contains none of the 14. "Not known" is not a publishable state. | FR-DSC-07, FR-CAT-04 | US-010, US-029 |
| BR-012 | The allergens of an ordered line are the item's own allergens plus those of every selected choice. A menu row shows the item's own allergens and adds "+ je nach Auswahl" (depending on choice) when any available choice adds an allergen. | FR-DSC-07, FR-DSC-09, FR-CAT-05, FR-POS-09 | US-010, US-026, US-029 |
| BR-013 | Every change to an item's or choice's allergens or ingredients creates a new declaration version that a Business Admin must confirm. The confirmed version is visible to customers within 60 seconds. A cart line keeps the declaration version the customer saw; at checkout, a line whose version is no longer current must be reviewed and confirmed again before payment. | FR-ORD-12, FR-CAT-04 | US-020, US-029 |
| BR-014 | Only items marked active and visible in the app are shown; hidden items stay in the catalog and are excluded from search. A sold-out item is shown dimmed with "Ausverkauft" and cannot be added; "until end of day" resets at the location's next opening, "until reset" stays until a user resets it. A cart with a sold-out line cannot be checked out until the line is removed. | FR-DSC-06, FR-CAT-03, FR-POS-04 | US-009, US-028, US-030 |
| BR-015 | Prices are entered and shown in EUR including VAT. Every item, option choice and service has a tax category: Reduced (10%) or Standard (20%) at launch, or Exempt (0%) for a business that applies the small-business VAT exemption, whose receipts state the exemption instead of VAT amounts. Rates are effective-dated reference data and the rate in force on the order date applies. | FR-DSC-06, FR-PAY-05, FR-APT-01, FR-CAT-03, FR-ONB-06 | US-009, US-014, US-021, US-030, US-038 |

### Ordering and pickup (BR-016 to BR-025)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-016 | A customer holds one cart for one location at a time. Adding an item from another location asks "Neuen Warenkorb starten" or "Aktuellen Warenkorb behalten". A cart is discarded after 24 hours without activity. | FR-ORD-01 | US-011 |
| BR-017 | Pickup slots are 15 minutes long and start inside the location's ordering hours; the last slot starts 15 minutes before ordering hours end. The earliest slot is the first slot boundary at or after now plus the preparation lead time (default 20 minutes, configurable 5 to 120). Without pre-order items only same-day slots are offered; with pre-order items the earliest slot is now plus the largest item lead time and the latest is 7 days ahead. Slots are offered only while the business is Live and the location is not paused. | FR-ORD-02, FR-ORD-11, FR-CAT-02 | US-012, US-018, US-036 |
| BR-018 | Each pickup slot takes at most N orders per location (default 6, configurable 1 to 30). A slot is offered while orders in Placed, Accepted, Preparing or Ready plus active holds are fewer than N. | FR-ORD-02, FR-CAT-02 | US-012, US-036 |
| BR-019 | Choosing a pickup slot creates a hold on one unit of its capacity for 10 minutes. The hold is extended once by 5 minutes when a 3-D Secure challenge starts. It is released when the order is placed, the slot is changed, checkout is left or the hold expires. | FR-ORD-03, FR-ORD-12 | US-012, US-020 |
| BR-020 | When payment authorization succeeds, slot capacity, item availability, prices and allergen versions are re-checked in one database transaction. If the slot is full, the customer is offered the next 3 free slots for 3 minutes and the authorization is voided if none is chosen. An order is never created in a full slot. | FR-ORD-12 | US-020 |
| BR-021 | Order status runs Placed to Accepted to Preparing to Ready to Completed; Placed can end in Rejected; Placed or Accepted can end in Canceled; Ready can end in Not collected. Only the business, or a system timeout, changes a status; the customer app itself never moves an order forward. | FR-ORD-05, FR-ORD-06, FR-ORD-09, FR-POS-03 | US-015, US-027 |
| BR-022 | An order must be accepted or rejected within 5 minutes of placement, or for a pre-order placed while the location is closed within 30 minutes of its next opening; otherwise it is auto-rejected with the reason "Keine Antwort vom Geschäft". A rejection needs a reason: Item sold out, Too busy, Closing early, or Other with text; the reason is shown to the customer. | FR-ORD-07, FR-POS-01, FR-POS-02 | US-015, US-026 |
| BR-023 | A customer can cancel while the order is Placed, or while it is Accepted and not yet Preparing until the location's cancellation cutoff (default 30 minutes before the slot starts, configurable 0 to 120). A Placed order's authorization is voided; an Accepted order is refunded in full automatically. Never after Preparing. | FR-ORD-08, FR-PAY-04, FR-CAT-02 | US-016, US-032, US-036 |
| BR-024 | An order is Completed only when staff match the 4-character pickup code (typed or scanned as QR); a pay-at-pickup order also needs staff to confirm "Bezahlt an der Kassa", the payment itself being taken on the business's own cash register. Staff can mark a Ready order Not collected from 60 minutes after its slot ends; prepaid amounts are not refunded automatically. | FR-ORD-05, FR-PAY-03, FR-POS-03, FR-POS-08 | US-013, US-015, US-027 |
| BR-025 | Rebook copies items, choices and quantities only, at current menu prices; unavailable items are skipped and listed. Slot, note, voucher and payment method are not copied. An open cart for another location triggers the BR-016 prompt. | FR-ORD-10 | US-017 |

### Appointments (BR-026 to BR-031)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-026 | A start time s is offered only if it lies on the 15-minute grid, s is at least now plus the minimum notice (default 2 hours, configurable 0 to 48), the date is within the booking horizon (default 8 weeks, configurable 1 to 12), and the interval [s, s + duration + buffer) lies inside one working interval of the staff member and overlaps no occupied interval (appointments including their buffers, breaks, absences). Intervals are half-open, so touching ends do not overlap. | FR-APT-02, FR-CAT-06 | US-021, US-022, US-031 |
| BR-027 | With "Beliebig" (any available) the customer sees the union of start times of all staff qualified for the service. At confirmation the system assigns the qualified free staff member with the fewest booked minutes that day; ties go to the staff display order. | FR-APT-02, FR-APT-04 | US-021, US-023 |
| BR-028 | Selecting a start time holds it for 10 minutes. Confirmation re-validates in the database, where a constraint rejects overlapping occupied intervals for the same staff member; if the time was lost, the customer is shown the 3 nearest free start times. | FR-APT-03, FR-APT-04 | US-023 |
| BR-029 | A customer can cancel or reschedule free of charge in the app until 24 hours before the start (configurable 2 to 48 hours); after that the app shows the business's phone number. No cancellation or no-show fees are charged in R1. | FR-APT-06 | US-024 |
| BR-030 | Appointment status runs Confirmed to Checked in to Completed; Confirmed can end in Canceled (by customer or business) or No-show, and No-show can be set only from 15 minutes after the start. | FR-APT-07, FR-POS-06 | US-025 |
| BR-031 | A change to working hours, breaks or absences that overlaps confirmed appointments cannot be saved until each one is reassigned to a qualified free staff member or canceled with a reason. Every business-initiated cancellation needs a reason and notifies the customer at once with a link to book again. | FR-APT-10, FR-CAT-08, FR-POS-06 | US-025, US-031 |

### Payments (BR-032 to BR-038)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-032 | In-app payment is by Visa or Mastercard card, Apple Pay or Google Pay through the card payment provider, with 3-D Secure 2 strong customer authentication; the platform never receives or stores card numbers. Pay at pickup is offered only where the business enables it. Appointments are paid at the venue in R1. | FR-PAY-01, FR-PAY-03, FR-APT-09 | US-013, US-023 |
| BR-033 | A payment is authorized at placement, captured when the business accepts, voided on rejection, auto-rejection or cancellation before acceptance, and refunded after capture. Provider webhooks are verified and processed once per event ID. | FR-PAY-02, FR-PAY-09, FR-POS-02 | US-013, US-026 |
| BR-034 | Refunds may be full or partial; the total refunded never exceeds the captured amount. A reason is required. A partial refund is allocated to VAT-rate groups in proportion to the paid gross per group, using the BR-035 rounding. Only a Business Admin, or the system on a customer cancellation, can refund. | FR-PAY-04 | US-016, US-032 |
| BR-035 | Prices are gross. An order-level discount is allocated to VAT-rate groups in proportion to each group's gross, rounded half up to the cent, with the highest-rate group taking the remainder so the parts add up exactly. VAT per group = gross after discount x rate / (100 + rate), rounded half up to the cent; net = gross - VAT. VAT is computed per rate group, not per line. | FR-ORD-04, FR-PAY-05, FR-ONB-06 | US-014, US-038 |
| BR-036 | One voucher code per order. A voucher is a fixed amount or a percentage, has a validity window, a total and a per-customer redemption limit and a minimum order value measured on the basket gross before discount. The discount never exceeds the basket gross minus EUR 0.50. A voucher is funded either by the business or by the platform. | FR-ORD-04, FR-PAY-06, FR-CAT-09, FR-ONB-09 | US-014, US-030, US-038 |
| BR-037 | Commission is 8% of the paid gross after business-funded discounts (a platform-funded discount does not reduce the base and is reimbursed to the business), rounded half up to the cent, plus 20% Austrian VAT on the commission. It is deducted at capture, reversed pro rata on refunds, and the card payment provider pays out the balance weekly. | FR-PAY-08 | US-032 |
| BR-038 | The receipt is issued in the business's name with its address, UID number where it has one, date, items and quantities, gross amount per VAT rate, and VAT rate and amount, which is the content of a small-amount invoice under section 11(6) UStG. In-app payment is therefore limited to orders of EUR 400.00 gross or less. | FR-PAY-07 | US-019 |

### Business portal (BR-039 to BR-041)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-039 | A Business Admin can manage everything for their own business. Staff see only the POS view (order board, sold-out toggles, pause, today's appointments) and cannot change catalog, prices, allergens, staff, refunds, statements or settings. Platform Admins cannot place orders or edit a business's catalog. | FR-CAT-07, FR-POS-07 | US-026, US-031 |
| BR-040 | A new order alerts on the order board, by sound and visually, until it is accepted or rejected. Staff can pause new orders for 15, 30 or 60 minutes or until resumed; no slots are offered during a pause and orders already placed are unaffected. If no order board session is connected for a location for 2 minutes while it accepts orders, ordering pauses automatically and the Business Admin is emailed. | FR-POS-01, FR-POS-05 | US-026, US-028 |
| BR-041 | Transactional messages (order confirmation and status changes, appointment confirmation, reminder 24 hours before the start, changes made by the business) are always sent by email and by push where enabled; marketing needs an opt-in. Push and email texts never contain allergen or other health information. | FR-ACC-06, FR-ORD-06, FR-APT-05 | US-005, US-015, US-024 |

### Onboarding and platform (BR-042 to BR-045)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-042 | A business application needs legal name, legal form, Firmenbuch number (companies) or GISA number (sole traders), UID number where VAT-registered (a small business without one confirms the VAT exemption), registered address, category and business types, at least one location address, a contact person with phone, and acceptance of the business terms. | FR-ONB-01, FR-ONB-02 | US-033, US-034 |
| BR-043 | A business goes Live only when verification is Approved, the card payment provider onboarding is Verified (food businesses, which take in-app payment), the legal information is complete, at least one location has ordering or working hours, and at least one item with a complete allergen declaration or one online-bookable service with a bookable staff member is published. | FR-CAT-01, FR-ONB-02, FR-ONB-03, FR-ONB-04 | US-034, US-035, US-036 |
| BR-044 | A suspension needs a statement of reasons sent to the Business Admin. The business leaves discovery at once; open orders run to completion; future appointments stay valid unless the Platform Admin cancels them, in which case each customer is notified. Ending the relationship for reasons other than a serious breach needs 30 days' notice. | FR-ONB-05 | US-037 |
| BR-045 | Allergen declarations, price changes, refunds, verification decisions, role changes and suspensions are written to an append-only audit trail with actor, before and after values and timestamp, retained for 7 years. | FR-ONB-07 | US-038 |

## 3. Decision tables

### 3.1 Allergen display, configured lines and the allergen filter

Implements BR-010 to BR-013 and FR-DSC-06, FR-DSC-07, FR-DSC-09, FR-CAT-04, FR-POS-09 and FR-ORD-12.

**Table 3.1a: what a configured line contains (hit policy Collect)**

| # | Source | Allergens added to the line |
|---|---|---|
| D1 | The item's own confirmed declaration | Its letters |
| D2 | Each selected option choice | That choice's letters |
| D3 | A choice that is available but not selected | None (displayed beside that choice only) |

The line's allergens are the union of D1 and D2. The order ticket on the board shows this union (FR-POS-09), not the item's own letters.

**Table 3.1b: menu marking when the customer's allergen filter is on (hit policy First, evaluated per item on the device)**

| # | Item's own allergens include a filtered allergen | An available choice adds a filtered allergen | Marking on the menu row | Text (de-AT) |
|---|---|---|---|---|
| M1 | Yes | Any | Item dimmed; letters of the filtered allergens in bold; still openable | Enthält H |
| M2 | No | Yes | Item not dimmed; caution label; in Item Details the choices that add the allergen are marked | Je nach Auswahl: H |
| M3 | No | No | No label. The filter never says "safe" or "allergen-free" | n/a |

A banner above the filtered menu reads "Filter: ohne H · Angaben des Betriebs. Spuren werden nicht erfasst." ("Filter: without H · information from the business. Traces are not recorded."). Filter selections never leave the device (FR-DSC-09).

**Table 3.1c: allergen version check at checkout (hit policy First, CR-005, R1.1)**

| # | Declaration version stored on the cart line | Result |
|---|---|---|
| V1 | Equal to the current confirmed version for the item and every selected choice | No action |
| V2 | Differs, and allergens were added | Line expanded; added letters shown in bold; the customer must tick "Neue Allergenangaben gelesen" before payment |
| V3 | Differs, and allergens were only removed, or only ingredients changed | Line expanded with the change; the customer must confirm before payment |
| V4 | The item or a selected choice is no longer published | Line marked unavailable; must be removed |

The check runs when the customer presses the order button and again when payment authorization succeeds (BR-020). An open pre-order whose item changes allergens after placement triggers an email to the customer. The customer can then cancel free of charge until preparation starts (TBD-12, closed by CR-005).

**Example.** At 08:52 on 2026-08-14 Konditorei Pomeranze changed "Topfentorte (Stück)" from A, C, G to A, C, G, H (hazelnut brittle added). A cart line created at 08:40 stores version 3 (A, C, G). At checkout the current version is 4, so V2 applies: the customer sees "H Schalenfrüchte" in bold and must confirm. Before CR-005, the same customer could pay without seeing the change, which is what happened in [INC-2026-006](../07-operations/incidents/INC-2026-006-stale-allergen-information.md).

### 3.2 Pickup slot offering

Implements BR-017, BR-018 and BR-040, and FR-ORD-02, FR-ORD-11 and FR-POS-05. Hit policy First, evaluated per candidate slot every time the slot list is requested.

| # | Condition | Result |
|---|---|---|
| P1 | Business not Live, or location paused (manually or automatically, BR-040) | No slots; message with the next time ordering opens |
| P2 | Slot does not start inside the ordering hours, or starts later than 15 minutes before ordering hours end | Not offered |
| P3 | Slot starts before the first slot boundary at or after now plus the lead time (with pre-order items: now plus the largest item lead time) | Not offered |
| P4 | Cart has no pre-order items and the slot is not today; or the slot is more than 7 days ahead | Not offered |
| P5 | Orders in Placed, Accepted, Preparing or Ready plus active holds are at least the slot capacity | Shown as "ausgebucht" and not selectable |
| P6 | Otherwise | Offered |

**Examples (Café Mohnblume, lead time 20 minutes, capacity 6, ordering hours 07:30 to 17:30).**

| Now | Earliest pickup | First slot offered | Rule |
|---|---|---|---|
| 11:52 | 12:12 | 12:15 to 12:30 | P3 |
| 11:55 | 12:15 | 12:15 to 12:30 (boundary included) | P3 |
| 11:56 | 12:16 | 12:30 to 12:45 | P3 |
| 11:52, slot 12:15 holds 5 orders and 1 active hold | 12:12 | 12:30 to 12:45; 12:15 shown as "ausgebucht" | P5 |
| 17:00 | 17:20 | None today ("Heute sind keine Abholzeiten mehr frei."); the last slot would start at 17:15 | P2, P3 |
| Cart contains "Schokoladentorte, ganz" (lead time 48 h), now Monday 10:00 | Wednesday 10:00 | Wednesday 10:00 to 10:15, up to Monday 7 days ahead | P3, P4 |

### 3.3 Re-check when payment authorization succeeds

Implements BR-019, BR-020 and BR-033, and FR-ORD-12 (CR-006, R1.1). Hit policy First, in one database transaction that locks the slot's capacity row.

| # | Condition at authorization | Outcome | Authorisation |
|---|---|---|---|
| R1 | A line is sold out or no longer published | Back to checkout with the line marked | Voided |
| R2 | A line's price or allergen version changed since the order button was pressed | Back to checkout showing the change (table 3.1c) | Voided; the customer confirms and pays again |
| R3 | The customer's hold is still active, or the slot still has a free unit | Order created in status Placed | Kept for capture at acceptance |
| R4 | The slot is full and at least one later slot today is free | The next 3 free slots are offered and held for 3 minutes | Kept; voided if no slot is chosen in 3 minutes |
| R5 | The slot is full and no later slot is free today | "Heute sind keine Abholzeiten mehr frei." | Voided |

**Hold extension (BR-019).** When the card payment provider reports that a 3-D Secure challenge has started, the hold is extended once by 5 minutes, so it lasts 15 minutes at most. Example: the hold is created at 11:40:00 and would expire at 11:50:00. The challenge starts at 11:43:10, so the hold now expires at 11:55:00. The bank confirms at 11:56:20, after the hold has expired. If another order took the last unit at 11:55:30, R4 applies and the customer chooses 12:30 from the offered slots. Before CR-006 the order was created in the full slot, which is what happened in [INC-2026-011](../07-operations/incidents/INC-2026-011-pickup-slots-over-capacity.md).

### 3.4 VAT, discounts, commission and refunds

Implements BR-015 and BR-034 to BR-037, and FR-PAY-04 to FR-PAY-06, FR-PAY-08 and FR-ONB-06.

**Method (BR-035).**
1. Group the lines by VAT rate and sum their gross amounts (item price plus choice price changes, times quantity).
2. Allocate an order-level discount to the groups in proportion to their gross, rounded to the cent, in ascending rate order; the highest-rate group takes the remainder.
3. For each group: VAT = gross after discount x rate / (100 + rate), rounded to the cent; net = gross after discount - VAT.
4. A business with the small-business exemption uses rate 0% for every line; its receipt shows no VAT amounts and states "Umsatzsteuerbefreit, Kleinunternehmer".

**Table 3.4a: voucher eligibility (hit policy First, BR-036)**

| # | Condition | Result |
|---|---|---|
| G1 | Code does not exist | Rejected: "Diesen Gutscheincode gibt es nicht." |
| G2 | Today outside the validity window | Rejected: "Dieser Gutschein ist derzeit nicht gültig." |
| G3 | Total or per-customer redemption limit reached | Rejected: "Dieser Gutschein wurde bereits eingelöst." |
| G4 | Business-funded voucher of another business | Rejected: "Dieser Gutschein gilt nicht für {Geschäft}." |
| G5 | Basket gross before discount below the minimum order value | Rejected: "Dieser Gutschein gilt ab einem Bestellwert von € {Betrag}." |
| G6 | Otherwise | Applied: discount = fixed value, or percentage x basket gross rounded; never more than basket gross - EUR 0.50 |

**Worked example (canonical, US-014, TC-PAY-003).** Café Mohnblume, voucher MOHN3: EUR 3.00 off from EUR 15.00, business-funded, valid 2026-09-01 to 2026-12-31, once per customer.

| Step | 10% group | 20% group | Total |
|---|---|---|---|
| Lines | 2 x Topfengolatsche EUR 3.40 = EUR 6.80; 1 x Schinken-Käse-Sandwich EUR 6.90 | 1 x Hausgemachte Limonade 0,5 l (EUR 3.90 + EUR 1.10) = EUR 5.00 | |
| Gross before discount | EUR 13.70 | EUR 5.00 | EUR 18.70 |
| Voucher eligible (G6) | | | EUR 18.70 is at least EUR 15.00 |
| Discount allocated | 3.00 x 13.70 / 18.70 = 2.1979, so EUR 2.20 | Remainder EUR 0.80 | EUR 3.00 |
| Gross after discount | EUR 11.50 | EUR 4.20 | EUR 15.70 |
| VAT | 11.50 x 10 / 110 = 1.0455, so EUR 1.05 | 4.20 x 20 / 120 = EUR 0.70 | EUR 1.75 |
| Net | EUR 10.45 | EUR 3.50 | EUR 13.95 |

Boundary: the same voucher on a basket of EUR 14.99 is rejected by G5; on exactly EUR 15.00 it is applied.

**Commission and payout (BR-037).** Commission base EUR 15.70 (the business-funded discount reduces the base). Commission 8% = 1.256, so EUR 1.26; VAT 20% on the commission EUR 0.25; total deducted EUR 1.51; payout EUR 14.19. If the voucher had been platform-funded, the base would be EUR 18.70 and the platform would reimburse the EUR 3.00 to the business in the weekly payout.

**Partial refund (BR-034), illustrative.** After capture, the business refunds EUR 5.00 because the lemonade was made with the wrong flavor and a pastry was missing.

| | 10% group | 20% group | Total |
|---|---|---|---|
| Paid gross | EUR 11.50 | EUR 4.20 | EUR 15.70 |
| Refund allocated pro rata | 5.00 x 11.50 / 15.70 = 3.6624, so EUR 3.66 | Remainder EUR 1.34 | EUR 5.00 |
| VAT reversed | 3.66 x 10 / 110 = EUR 0.33 | 1.34 x 20 / 120 = EUR 0.22 | EUR 0.55 |
| Commission reversed | 1.26 x 5.00 / 15.70 = 0.4013, so EUR 0.40, plus EUR 0.08 VAT | | EUR 0.48 |

### 3.5 Appointment start-time eligibility

Implements BR-026 to BR-028 and FR-APT-02 to FR-APT-04. Hit policy Collect: a candidate start time s, on the 15-minute grid from the start of each working interval, is offered only if every check passes. D = service duration + buffer after.

| # | Check | Passes when |
|---|---|---|
| E1 | Notice | s is at least now + minimum notice (default 2 h) |
| E2 | Horizon | The date of s is at most today + booking horizon (default 8 weeks) |
| E3 | Working time | [s, s + D) lies inside one working interval of the staff member (breaks split a shift into intervals) |
| E4 | Appointments | [s, s + D) does not overlap the occupied interval of any Confirmed or Checked-in appointment of that staff member, buffers included |
| E5 | Absences | [s, s + D) does not overlap an absence |
| E6 | Holds | [s, s + D) does not overlap another customer's active hold for that staff member |
| E7 | Qualification | The staff member is qualified for the service and marked bookable |

With "Beliebig" (BR-027), the customer sees the union over all qualified staff. The assignment at confirmation takes the staff member with the fewest booked minutes that day among those for whom all checks still pass.

**Worked example (canonical, US-022, TC-APT-002).** Haarstudio Linde; Lena Auer works 09:00 to 13:00 on Tuesday 2026-10-13. One appointment exists at 10:30 (Herrenhaarschnitt, 30 + 15 minutes, occupying 10:30 to 11:15). Requested: Damenhaarschnitt, 45 + 15 = 60 minutes. Viewed on 2026-10-12 at 18:00.

| Start | Interval | E3 | E4 | Offered |
|---|---|---|---|---|
| 09:00 | 09:00 to 10:00 | Pass | Pass | Yes |
| 09:15 | 09:15 to 10:15 | Pass | Pass | Yes |
| 09:30 | 09:30 to 10:30 | Pass | Pass (touches 10:30) | Yes |
| 09:45 | 09:45 to 10:45 | Pass | Fail | No |
| 10:00 to 11:00 | various | Pass | Fail | No |
| 11:15 | 11:15 to 12:15 | Pass | Pass (touches 11:15) | Yes |
| 11:30 | 11:30 to 12:30 | Pass | Pass | Yes |
| 11:45 | 11:45 to 12:45 | Pass | Pass | Yes |
| 12:00 | 12:00 to 13:00 | Pass (touches 13:00) | Pass | Yes |
| 12:15 | 12:15 to 13:15 | Fail (buffer past 13:00) | Pass | No |
| 12:30, 12:45 | past 13:00 | Fail | Pass | No |

Result: 7 start times (09:00, 09:15, 09:30, 11:15, 11:30, 11:45, 12:00); capacity for 2 more appointments. After Sophie books 09:30: 4 start times (11:15, 11:30, 11:45, 12:00).

**Notice boundary (E1).** On 2026-10-13 itself, viewed at 07:15 the list starts at 09:15; viewed at 07:16 it starts at 09:30.

**Horizon boundary (E2).** Viewed on 2026-10-12 with an 8-week horizon, 2026-12-07 is bookable and 2026-12-08 is not.

### 3.6 Cancellation by the customer

Implements BR-023, BR-029 and BR-033, and FR-ORD-08, FR-APT-06 and FR-PAY-04. Hit policy First.

**Table 3.6a: orders**

| # | Status | Time | Payment | Customer can cancel in the app | Money |
|---|---|---|---|---|---|
| O1 | Placed | Any | In-app, authorized | Yes | Authorisation voided; nothing charged |
| O2 | Placed | Any | Pay at pickup | Yes | None |
| O3 | Accepted (not Preparing) | Now is at or before slot start minus the cutoff | In-app, captured | Yes | Full refund issued automatically |
| O4 | Accepted (not Preparing) | Now is at or before slot start minus the cutoff | Pay at pickup | Yes | None |
| O5 | Accepted | Now is after slot start minus the cutoff | Any | No: "Bitte ruf {Geschäft} an." | The business may refund (BR-034) |
| O6 | Preparing or Ready | Any | Any | No | The business may refund |
| O7 | Completed, Rejected, Canceled, Not collected | Any | Any | No | n/a |

Example: slot 12:15, cutoff 30 minutes. An Accepted order can be canceled in the app at 11:45 and not at 11:46. At 11:46 the Booking Details show the business's phone number.

**Table 3.6b: appointments**

| # | Status | Time | Customer can cancel or reschedule in the app |
|---|---|---|---|
| A1 | Confirmed | Now is at or before start minus the cutoff (default 24 h) | Yes, free of charge |
| A2 | Confirmed | Now is after start minus the cutoff | No; the business's phone number is shown |
| A3 | Checked in, Completed, Canceled, No-show | Any | No |

Example: appointment Tuesday 2026-10-13 09:30. Cancel or reschedule in the app until Monday 2026-10-12 09:30; from 09:31 the phone number is shown. A reschedule is a new booking that replaces the old one atomically: if the new time cannot be confirmed, the old appointment stays.

### 3.7 Payment actions by order event

Implements BR-032 to BR-034, FR-PAY-02, FR-PAY-03 and FR-PAY-09. Hit policy Unique on event and payment state.

| # | Event | Payment state before | Action | Payment state after |
|---|---|---|---|---|
| K1 | Order placed, in-app | None | Authorise (with 3-D Secure where the issuer requires it) | Authorised |
| K2 | Order placed, pay at pickup | None | None | Due at pickup |
| K3 | Business accepts | Authorised | Capture the full amount; commission taken as an application fee | Captured |
| K4 | Business rejects, or auto-rejection | Authorised | Void | Voided |
| K5 | Customer cancels while Placed | Authorised | Void | Voided |
| K6 | Customer cancels while Accepted, within the cutoff | Captured | Refund in full | Refunded |
| K7 | Business refunds part of the amount | Captured or Partially refunded | Refund the amount; total refunded at most the captured amount | Partially refunded, or Refunded when the total is reached |
| K8 | Capture fails (for example the authorization was canceled by the issuer) | Authorised | Order canceled; customer and staff told "Zahlung fehlgeschlagen"; reconciliation alert | Failed |
| K9 | Staff hand over a pay-at-pickup order and confirm "Bezahlt an der Kassa" | Due at pickup | None in the platform; payment taken on the business's own register | Paid at counter |
| K10 | Duplicate webhook for an event ID already processed | Any | Ignored; 200 returned to the provider | Unchanged |

### 3.8 Go-live gate

Implements BR-043 and FR-ONB-02 to FR-ONB-04. Hit policy Collect: the Platform Admin's Publish action is enabled only when every applicable condition holds.

| # | Condition | Food | Salon and spa |
|---|---|---|---|
| L1 | Verification decision is Approved | Required | Required |
| L2 | Card payment provider onboarding is Verified | Required | Not required in R1 (paid at the venue) |
| L3 | Legal information complete: legal name, address, Firmenbuch or GISA number, UID number or exemption, phone, email | Required | Required |
| L4 | At least one location with ordering hours (food) or working hours for a bookable staff member (salon and spa) | Required | Required |
| L5 | At least one published item with a complete allergen declaration | Required | n/a |
| L6 | At least one online-bookable service with a qualified, bookable staff member | n/a | Required |

Example: Massageatelier Ruhepol was Approved on 2026-06-08, but Publish stayed disabled until 2026-06-10 because L6 failed: its only therapist had no working hours yet. Payment onboarding (L2) does not apply to it.

### 3.9 Notifications

Implements BR-041, FR-ORD-06, FR-APT-05 and FR-APT-10. Hit policy Collect.

| Event | Recipient | Push (if enabled) | Email | Timing |
|---|---|---|---|---|
| Order placed | Customer | Yes | Yes (confirmation) | Immediately |
| Accepted, delayed, Ready | Customer | Yes | No | Immediately |
| Rejected or auto-rejected | Customer | Yes | Yes | Immediately |
| Receipt available | Customer | No | Yes | After capture |
| Appointment confirmed or rescheduled | Customer | Yes | Yes, with calendar file | Immediately |
| Appointment reminder | Customer | Yes | Yes | 24 h before start; not sent if booked less than 24 h ahead |
| Appointment canceled by the business | Customer | Yes | Yes, with reason and rebook link | Immediately |
| Allergens changed on an open pre-order | Customer | No | Yes | Within 5 minutes of the change |
| Order board disconnected and ordering auto-paused | Business Admin | No | Yes | After 2 minutes (BR-040) |
| Statement of reasons (suspension) | Business Admin | No | Yes | At suspension |

Push titles and email subjects carry the business name and the status only. They never include allergens or health information (BR-041).

## 4. Rules changed by change requests

| Rule | Change | CR | SRS version | Origin |
|---|---|---|---|---|
| BR-017 | Pre-order items with a lead time up to 72 hours and a 7-day horizon | CR-001 | v1.1 | Bakery pilots (whole cakes) |
| BR-015 | Exempt (0%) category for businesses with the small-business VAT exemption | UAT clarification | v1.2 | UAT finding at Kaffeebar Zimtstern |
| BR-040 | Automatic pause after 2 minutes without a connected order board | UAT clarification | v1.2 | UAT script UAT-06 |
| BR-013 | Declaration versions, 60-second freshness, re-confirmation at checkout, email on open pre-orders | CR-005 | v1.3 | INC-2026-006 |
| BR-019 | One 5-minute hold extension when a 3-D Secure challenge starts | CR-006 | v1.3 | INC-2026-011 |
| BR-020 | New rule: re-check capacity, availability, prices and allergens at authorization; never create an order in a full slot | CR-006 | v1.3 | INC-2026-011 |
| BR-024, BR-032 | Confirmed unchanged when CR-003 (counter payments in the POS view) was rejected | CR-003 | n/a | Cash register rules |
| BR-029 | Confirmed unchanged when CR-004 (automatic no-show fees) was rejected | CR-004 | n/a | Payment and consumer rules |
| BR-008 | Confirmed unchanged when CR-007 (public star ratings) was rejected | CR-007 | n/a | Neutral ranking principle |

## 5. Rule precedence and known tensions

| # | Tension | Resolution in R1 |
|---|---|---|
| 1 | BR-018 (capacity) vs BR-019 (holds expire) during a slow 3-D Secure challenge | BR-020 decides at authorization; the database never accepts an order into a full slot (table 3.3). |
| 2 | BR-022 (5-minute acceptance) vs pre-orders placed in the evening | Pre-orders placed while closed must be accepted within 30 minutes of the next opening. |
| 3 | BR-023 (customer cancels Accepted orders) vs the kitchen having started | Cancellation stops at Preparing, whatever the cutoff says. |
| 4 | BR-013 (versions) vs NFR-PERF-01 (fast menus) | Menus are not cached by shared caches for more than 60 seconds; images still are (NFR-DAT-02). |
| 5 | BR-026 (buffer must fit the shift) vs salons that clean after closing | The buffer is part of paid working time; a salon that wants the last start later extends the shift. |
| 6 | BR-029 (free cancellation) vs no-show costs | No fees in R1 (CR-004 rejected); reminders carry OBJ-02. |
| 7 | BR-035 (VAT per group) vs a receipt line that shows its own gross | Lines show gross amounts; VAT is shown only per rate group, as the method requires. |
| 8 | BR-005 (erasure) vs BR-045 (7-year audit) | Audit events reference a pseudonymous customer ID; the name and contact details are erased. |
| 9 | BR-044 (suspend at once) vs open orders | Open orders run to completion so customers are not left without food they paid for. |
| 10 | BR-011 (allergen gate) vs drinks such as water | "Contains none of the 14" is a declaration and must be confirmed like any other. |

## Related documents

- [Software Requirements Specification](SRS.md)
- [Business Requirements Document](BRD.md)
- [Non-functional requirements](non-functional-requirements.md)
- [Compliance mapping](compliance-mapping.md)
- [Glossary](glossary.md)
- [Requirements traceability matrix](requirements-traceability-matrix.md)
- [State machines](../03-design/diagrams/state-machines.md)
- [Change request log](../05-delivery/change-request-log.md)
- [Test cases](../06-quality/test-cases.md)
