# Entity Relationship Diagram: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | LC-DES-ERD |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst (co-authored with the Engineering Lead) |
| Last updated | 2026-09-25 |
| Reviewers | Engineering Lead; QA Lead; Compliance Lead; Finance Lead |

**Purpose and scope.** The logical data model of the platform in PostgreSQL, split into four subject areas so each diagram stays readable. Field-level definitions, classifications and retention are in the [data dictionary](data-dictionary.md). Holds are not tables; they live in Redis with a time-to-live ([ADR-002](../architecture/adr/ADR-002-slot-holds-redis-database-guard.md)).

Conventions:
- Primary keys are UUIDs (`id`). Human-readable keys (`B-1001`, `L-1001`, `QO-2026-004218`) are separate unique columns.
- Every business-scoped table carries `business_id` and is protected by row-level security (NFR-SEC-05).
- Money is stored as integer cents (`*_cents`); timestamps as `timestamptz` in UTC.
- Rows that carry legal or financial meaning are never updated in place: allergen declarations, status events, payments, refunds and audit events.

## 1. Customers and consent

```mermaid
erDiagram
  CUSTOMER ||--o{ CONSENT : "gives"
  CUSTOMER ||--o{ SAVED_LOCATION : "saves"
  CUSTOMER ||--o{ DEVICE_TOKEN : "registers"
  CUSTOMER ||--o| DELETION_REQUEST : "may request"
  CUSTOMER ||--o{ ORDER : "places"
  CUSTOMER ||--o{ APPOINTMENT : "books"
  LOCATION ||--o{ SAVED_LOCATION : "is saved as"
  CUSTOMER {
    uuid id PK
    text email UK "lower-case"
    text mobile_e164 UK
    text first_name
    text last_name
    text password_hash "Argon2id, null for social-only"
    text locale "de-AT or en"
    boolean age_14_declared
    text status "Active, Locked, Deactivated, Pseudonymised"
    timestamptz created_at
  }
  CONSENT {
    uuid id PK
    uuid customer_id FK
    text purpose "terms, privacy, marketing_email, marketing_push, analytics"
    text document_version
    boolean granted
    text source "sign_up, profile"
    timestamptz recorded_at
  }
  SAVED_LOCATION {
    uuid customer_id PK
    uuid location_id PK
    timestamptz saved_at
  }
  DEVICE_TOKEN {
    uuid id PK
    uuid customer_id FK
    text platform "ios, android"
    text token
    boolean push_enabled
  }
  DELETION_REQUEST {
    uuid id PK
    uuid customer_id FK
    timestamptz requested_at
    timestamptz completed_at
  }
```

## 2. Businesses, catalog and allergens

```mermaid
erDiagram
  BUSINESS ||--|{ LOCATION : "operates"
  BUSINESS ||--o{ BUSINESS_USER : "employs"
  BUSINESS ||--o{ MENU_CATEGORY : "defines"
  BUSINESS ||--o{ ITEM : "sells"
  LOCATION ||--o{ OPENING_INTERVAL : "has"
  LOCATION ||--o{ CLOSURE : "has"
  MENU_CATEGORY ||--o{ ITEM : "groups"
  ITEM ||--o{ OPTION_GROUP : "offers"
  OPTION_GROUP ||--|{ OPTION_CHOICE : "contains"
  ITEM ||--|{ ALLERGEN_DECLARATION : "is declared by"
  OPTION_CHOICE ||--|{ ALLERGEN_DECLARATION : "is declared by"
  ITEM ||--o{ ITEM_LOCATION_STATE : "has per location"
  LOCATION ||--o{ ITEM_LOCATION_STATE : "tracks"
  TAX_CATEGORY ||--|{ TAX_RATE : "has effective-dated"
  TAX_CATEGORY ||--o{ ITEM : "applies to"
  BUSINESS {
    uuid id PK
    text business_key UK "B-1001"
    text legal_name
    text legal_form
    text register_number "FN or GISA"
    text uid_number "ATU plus 8 digits, nullable"
    boolean vat_exempt_small_business
    text category "FOOD, SALON, SPA"
    text status "Applied to Closed"
    text brand_colour_hex
    timestamptz live_at
  }
  LOCATION {
    uuid id PK
    uuid business_id FK
    text location_key UK "L-1001"
    text address
    geography geo_point "PostGIS"
    int slot_capacity "1 to 30"
    int lead_time_min "5 to 120"
    int cancel_cutoff_min "0 to 120"
    boolean pay_at_pickup
    timestamptz paused_until
  }
  BUSINESS_USER {
    uuid id PK
    uuid business_id FK
    text email UK
    text role "BIZ-ADM, BIZ-STF"
    boolean bookable
    text status
  }
  ITEM {
    uuid id PK
    uuid business_id FK
    uuid category_id FK
    text name_de
    text name_en
    int price_cents "VAT-inclusive"
    text tax_category FK
    int lead_time_h "0 to 72"
    boolean visible_in_app
    text publication "Draft, Published, Hidden"
    int current_declaration_version
  }
  OPTION_CHOICE {
    uuid id PK
    uuid option_group_id FK
    text name_de
    int price_delta_cents
    int current_declaration_version
  }
  ALLERGEN_DECLARATION {
    uuid id PK
    uuid item_id FK "or choice"
    uuid choice_id FK "or item"
    int version
    text allergen_codes "letters, empty when none"
    boolean contains_none_confirmed
    uuid confirmed_by FK
    timestamptz confirmed_at
  }
  ITEM_LOCATION_STATE {
    uuid item_id PK
    uuid location_id PK
    text sold_out "No, UntilEndOfDay, UntilReset"
    timestamptz sold_out_at
  }
  TAX_RATE {
    text tax_category PK
    date valid_from PK
    numeric rate_percent
  }
```

## 3. Orders, payments and receipts

```mermaid
erDiagram
  CUSTOMER ||--o{ ORDER : "places"
  LOCATION ||--o{ ORDER : "fulfills"
  ORDER ||--|{ ORDER_LINE : "contains"
  ORDER_LINE ||--o{ ORDER_LINE_CHOICE : "configured with"
  ORDER_LINE }o--|| ALLERGEN_DECLARATION : "sold with version"
  ORDER ||--|{ ORDER_STATUS_EVENT : "moves through"
  ORDER ||--o| PAYMENT : "is paid by"
  PAYMENT ||--o{ REFUND : "is refunded by"
  ORDER ||--o| RECEIPT : "is evidenced by"
  ORDER ||--|{ ORDER_VAT_GROUP : "is taxed by"
  VOUCHER ||--o{ VOUCHER_REDEMPTION : "is redeemed in"
  ORDER ||--o| VOUCHER_REDEMPTION : "uses"
  LOCATION ||--o{ PICKUP_SLOT_COUNTER : "limits"
  PAYMENT ||--o{ WEBHOOK_EVENT : "is updated by"
  ORDER {
    uuid id PK
    text order_ref UK "QO-2026-004218"
    uuid customer_id FK
    uuid location_id FK
    uuid business_id FK
    timestamptz slot_start
    text status "Placed to Not collected"
    text pickup_code "4 characters"
    text payment_mode "in_app, pay_at_pickup"
    int gross_before_discount_cents
    int discount_cents
    int total_cents
    timestamptz accept_deadline
  }
  ORDER_LINE {
    uuid id PK
    uuid order_id FK
    uuid item_id FK
    int quantity "1 to 20"
    int unit_price_cents
    text tax_category
    int declaration_version
    text line_allergen_codes "item plus choices"
    text note
  }
  ORDER_VAT_GROUP {
    uuid order_id PK
    numeric rate_percent PK
    int gross_cents
    int discount_cents
    int vat_cents
    int net_cents
  }
  PICKUP_SLOT_COUNTER {
    uuid location_id PK
    timestamptz slot_start PK
    int active_orders
    int capacity_snapshot
  }
  PAYMENT {
    uuid id PK
    uuid order_id FK
    text provider_payment_ref UK
    text state "Authorised, Captured, Voided, Refunded"
    int amount_cents
    int commission_cents
    int commission_vat_cents
    timestamptz captured_at
  }
  REFUND {
    uuid id PK
    uuid payment_id FK
    int amount_cents
    text reason
    uuid issued_by FK
    timestamptz created_at
  }
  VOUCHER {
    uuid id PK
    text code UK
    text funded_by "business, commerce"
    uuid business_id FK "null when platform-funded"
    text kind "fixed, percent"
    int value
    int min_order_cents
    int max_redemptions
    int max_per_customer
    date valid_from
    date valid_to
  }
  WEBHOOK_EVENT {
    text provider_event_id PK
    text type
    timestamptz received_at
    timestamptz processed_at
  }
```

## 4. Appointments, onboarding and audit

```mermaid
erDiagram
  BUSINESS ||--o{ SERVICE : "offers"
  SERVICE ||--o{ SERVICE_STAFF : "performed by"
  BUSINESS_USER ||--o{ SERVICE_STAFF : "qualified for"
  BUSINESS_USER ||--o{ WORKING_INTERVAL : "works"
  BUSINESS_USER ||--o{ ABSENCE : "is absent"
  BUSINESS_USER ||--o{ APPOINTMENT : "serves"
  SERVICE ||--o{ APPOINTMENT : "is booked as"
  CUSTOMER ||--o{ APPOINTMENT : "books"
  APPOINTMENT ||--o| APPOINTMENT : "replaces (reschedule)"
  BUSINESS ||--|| BUSINESS_APPLICATION : "starts as"
  BUSINESS_APPLICATION ||--o{ VERIFICATION_DECISION : "is decided by"
  BUSINESS ||--o| PAYMENT_ACCOUNT : "is paid out through"
  BUSINESS ||--o{ SUSPENSION : "may receive"
  BUSINESS ||--o{ AUDIT_EVENT : "is audited by"
  SERVICE {
    uuid id PK
    uuid business_id FK
    text name_de
    int duration_min "15 to 480, step 5"
    int buffer_after_min "0 to 60"
    int price_cents
    boolean price_from
    text tax_category
    boolean online_bookable
  }
  WORKING_INTERVAL {
    uuid id PK
    uuid staff_id FK
    int weekday "1 to 7"
    time start_local
    time end_local
    date valid_from
  }
  APPOINTMENT {
    uuid id PK
    text appointment_ref UK
    uuid customer_id FK
    uuid service_id FK
    uuid staff_id FK
    timestamptz starts_at
    timestamptz occupied_until "start plus duration plus buffer"
    text status "Confirmed to No-show"
    text note "up to 300 characters"
    uuid replaces_id FK
  }
  BUSINESS_APPLICATION {
    uuid id PK
    uuid business_id FK
    timestamptz submitted_at
    text uid_check "valid, invalid, pending"
    text status
  }
  VERIFICATION_DECISION {
    uuid id PK
    uuid application_id FK
    text decision "approve, reject, request_info"
    text reason
    uuid decided_by FK
    timestamptz decided_at
  }
  PAYMENT_ACCOUNT {
    uuid business_id PK
    text provider_account_ref UK
    text kyc_status "Pending, Verified, Restricted"
  }
  SUSPENSION {
    uuid id PK
    uuid business_id FK
    text statement_of_reasons
    timestamptz starts_at
    timestamptz ends_at
  }
  AUDIT_EVENT {
    uuid id PK
    uuid business_id FK
    text actor_type
    uuid actor_id
    text action
    text entity
    jsonb before_value
    jsonb after_value
    timestamptz at
  }
```

Constraint of note: `APPOINTMENT` carries an exclusion constraint `EXCLUDE USING gist (staff_id WITH =, tstzrange(starts_at, occupied_until, '[)') WITH &&) WHERE (status IN ('Confirmed','CheckedIn'))`. The half-open range `[)` is what makes touching appointments valid (BR-026).

## Related documents

- [Data dictionary](data-dictionary.md)
- [System architecture](../architecture/system-architecture.md)
- [State machines](../diagrams/state-machines.md)
- [Business rules](../../02-requirements/business-rules.md)
