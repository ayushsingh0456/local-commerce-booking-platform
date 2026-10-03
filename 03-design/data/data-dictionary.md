# Data Dictionary: Quartier

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DES-DD |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Engineering Lead; Compliance Lead; Finance Lead; QA Lead |

**Purpose and scope.** Field-level definitions for the tables in the [ERD](erd.md) that carry business rules, personal data or money. Each table lists its fields, rules, data classification and retention. Technical columns (`created_at`, `updated_at`, `row_version`) exist on every table and are not repeated.

## 1. Classification and retention

| Class | Meaning | Examples | Handling |
|---|---|---|---|
| P1 Personal | Identifies a customer or a person at a business | Name, email, mobile, device token | Encrypted at rest; access by role; erased or pseudonymized on deletion (BR-005) |
| P2 Personal, sensitive by context | Could reveal health information | Order line allergens linked to a customer | Kept with the order for the business's food records; never used for profiling; never in push, email subjects, analytics or error events |
| F Financial | Amounts, payments, refunds, VAT, commission | `total_cents`, `vat_cents` | Retained 7 years (BAO section 132); immutable after creation |
| B Business | Business data, not personal | Menu, prices, opening hours | Retained while the business is active and 7 years after closure for legal information |
| A Audit | Who did what | `audit_events` | Append-only, 7 years (BR-045) |

| Data set | Retention | Trigger |
|---|---|---|
| Customer profile (P1) | Until deletion; erased or pseudonymized within 30 days of the request | FR-ACC-07 |
| Orders, payments, receipts (F, P2) | 7 years from the end of the calendar year of the order | BR-005 |
| Appointments | 3 years, then pseudonymized | Not a bookkeeping record; kept for disputes |
| Consent records | Lifetime of the account plus 3 years | Evidence of consent |
| Audit events (A) | 7 years | BR-045 |
| Webhook events | 13 months | Reconciliation |

## 2. Tables

### customers

| Field | Type | Rule | Class |
|---|---|---|---|
| email | text | Unique among customers; lower-case; not used by any portal account (BR-002) | P1 |
| mobile_e164 | text | Unique among customers; valid mobile number in E.164 | P1 |
| first_name, last_name | text | 2 to 50 characters; letters, spaces, hyphens, apostrophes | P1 |
| password_hash | text | Argon2id; null when the account uses only Google or Apple | P1 |
| age_14_declared | boolean | Must be true at creation; self-declared, not verified (BR-002) | P1 |
| locale | text | `de-AT` (default) or `en` | P1 |
| status | text | Active, Locked (until `locked_until`), Deactivated, Pseudonymised | P1 |
| failed_sign_ins, locked_until | int, timestamptz | Lock for 15 minutes after the fifth consecutive failure (BR-004) | P1 |

### consents

| Field | Type | Rule | Class |
|---|---|---|---|
| purpose | text | terms, privacy, marketing_email, marketing_push, analytics | P1 |
| document_version | text | Version of the terms or privacy notice, for example `terms-2026-06` | P1 |
| granted | boolean | One row per change; the latest row per purpose is the current state (BR-003) | P1 |

### businesses

| Field | Type | Rule | Class |
|---|---|---|---|
| business_key | text | `B-` plus 4 digits | B |
| legal_name, legal_form | text | Required (BR-042) | B |
| register_number | text | `FN 123456a` for registered forms, digits for GISA | B |
| uid_number | text | `ATU` plus 8 digits; null only when `vat_exempt_small_business` is true | B |
| vat_exempt_small_business | boolean | When true, all items use tax category Exempt (BR-015) | B |
| category | text | FOOD, SALON, SPA | B |
| status | text | Applied, InReview, InfoRequested, Approved, Live, Suspended, Rejected, Closed | B |
| brand_colour_hex | text | Accepted only with at least 4.5:1 contrast against white (FR-CAT-01) | B |

### locations

| Field | Type | Rule | Class |
|---|---|---|---|
| geo_point | geography | From the geocoding provider; used for distance (BR-006) | B |
| slot_capacity | int | 1 to 30, default 6 (BR-018) | B |
| lead_time_min | int | 5 to 120, default 20 (BR-017) | B |
| cancel_cutoff_min | int | 0 to 120, default 30 (BR-023) | B |
| min_notice_min, horizon_weeks, free_cancel_hours | int | 0 to 2,880 (default 120); 1 to 12 (default 8); 2 to 48 (default 24) (BR-026, BR-029) | B |
| pay_at_pickup | boolean | Offer pay at pickup (FR-PAY-03) | B |
| paused_until, pause_reason | timestamptz, text | Manual or automatic pause (BR-040) | B |

### items and option_choices

| Field | Type | Rule | Class |
|---|---|---|---|
| price_cents / price_delta_cents | int | VAT-inclusive; item 10 to 99,999 cents | B |
| tax_category | text | REDUCED_10, STANDARD_20, EXEMPT_0 (BR-015) | B |
| lead_time_h | int | 0 to 72; above 0 makes the item a pre-order item (BR-017) | B |
| publication | text | Draft, Published, Hidden; Published only with a confirmed declaration for the item and every choice (BR-011) | B |
| current_declaration_version | int | Points to the latest confirmed `allergen_declarations.version` (BR-013) | B |

### allergen_declarations

| Field | Type | Rule | Class |
|---|---|---|---|
| item_id or choice_id | uuid | Exactly one is set | B |
| version | int | Starts at 1; increases by 1 with every confirmed change; never updated in place (BR-013) | B |
| allergen_codes | text | Subset of A, B, C, D, E, F, G, H, L, M, N, O, P, R in that order (BR-010) | B |
| contains_none_confirmed | boolean | True only when `allergen_codes` is empty and the Business Admin confirmed it (BR-011) | B |
| confirmed_by, confirmed_at | uuid, timestamptz | Business Admin and time; also written to `audit_events` | A |

### orders

| Field | Type | Rule | Class |
|---|---|---|---|
| order_ref | text | `QO-` year `-` 6 digits | F |
| slot_start | timestamptz | Start of the 15-minute pickup slot (BR-017) | F |
| status | text | Placed, Accepted, Preparing, Ready, Completed, Rejected, Canceled, NotCollected (BR-021) | F |
| reject_reason | text | SoldOut, TooBusy, ClosingEarly, NoResponse (auto), Other with text (BR-022) | F |
| pickup_code | text | 4 characters from an alphabet without 0, O, 1, I, L; unique among the location's open orders that day | P1 |
| payment_mode | text | in_app, pay_at_pickup | F |
| gross_before_discount_cents, discount_cents, total_cents | int | Per BR-035; `total_cents` at most 40,000 for in_app (BR-038) | F |
| accept_deadline | timestamptz | Placement plus 5 minutes, or next opening plus 30 minutes for pre-orders placed while closed (BR-022) | F |
| delay_min_total | int | Sum of delays, at most 60 | F |

### order_lines and order_vat_groups

| Field | Type | Rule | Class |
|---|---|---|---|
| order_lines.declaration_version | int | Version the customer confirmed; re-checked at authorization (BR-013, BR-020) | F |
| order_lines.line_allergen_codes | text | Union of the item's and the selected choices' allergens (BR-012) | P2 |
| order_lines.quantity | int | 1 to 20 | F |
| order_vat_groups.rate_percent | numeric | Rate in force on the order date (BR-015) | F |
| order_vat_groups.discount_cents | int | Pro rata allocation, highest rate takes the remainder (BR-035) | F |
| order_vat_groups.vat_cents, net_cents | int | Rounded half up per group (BR-035) | F |

### payments and refunds

| Field | Type | Rule | Class |
|---|---|---|---|
| payments.state | text | Authorised, Captured, Voided, PartiallyRefunded, Refunded, Failed, DueAtPickup, PaidAtCounter (table 3.7) | F |
| payments.commission_cents | int | 8% of the commission base, rounded (BR-037) | F |
| payments.commission_vat_cents | int | 20% of `commission_cents`, rounded | F |
| refunds.amount_cents | int | Sum over a payment at most the captured amount (BR-034) | F |
| refunds.reason | text | Required | F |

### appointments

| Field | Type | Rule | Class |
|---|---|---|---|
| starts_at | timestamptz | On the 15-minute grid; at least now plus the minimum notice; within the horizon (BR-026) | P1 |
| occupied_until | timestamptz | `starts_at` plus duration plus buffer | B |
| status | text | Confirmed, CheckedIn, Completed, Canceled, NoShow (BR-030) | P1 |
| cancelled_by, cancel_reason | text | customer or business; reason required for business (BR-031) | P1 |
| replaces_id | uuid | Set on a reschedule; the old appointment is canceled in the same transaction | P1 |

### audit_events

| Field | Type | Rule | Class |
|---|---|---|---|
| action | text | For example `allergen.confirmed`, `price.changed`, `refund.issued`, `verification.decided`, `role.changed`, `business.suspended` (BR-045) | A |
| before_value, after_value | jsonb | Full values of the changed fields | A |
| actor_id | uuid | Pseudonymous ID; survives customer erasure | A |

## Related documents

- [ERD](erd.md)
- [Business rules](../../02-requirements/business-rules.md)
- [Compliance mapping](../../02-requirements/compliance-mapping.md)
- [Glossary](../../02-requirements/glossary.md)
