# API Guidelines and Error Catalog: Quartier

## Document control

| Field | Value |
|---|---|
| Document ID | QT-API-GL |
| Version | 1.3 |
| Status | Approved |
| Owner | Engineering Lead (co-authored with the Business Analyst) |
| Last updated | 2026-09-25 |
| Reviewers | Backend developers; mobile developers; web developer; QA Lead |

**Purpose and scope.** The conventions every Quartier endpoint follows, the error catalog with the user-facing messages, and the configuration endpoints that the [OpenAPI specification](openapi.yaml) does not list. The OpenAPI file is the contract. This document explains the choices behind it so that new endpoints stay consistent.

## 1. Surfaces and authentication

| Route group | Caller | Token | Notes |
|---|---|---|---|
| `/v1/...` | Customer Mobile App | Customer access token (15 minutes) and rotating refresh token (30 days) | Public reads (locations, search, catalog, availability) also work without a token |
| `/v1/business/...` | Business Web Portal and POS view | Portal token carrying `businessId`, `role` (BIZ-ADM or BIZ-STF) and `locationIds` | Staff tokens are accepted only on POS operations (FR-POS-07); anything else returns 403 `FORBIDDEN_FOR_ROLE` |
| `/v1/platform/...` | Platform Admin portal | Platform token issued after a second factor | Never accepted on customer or business routes |
| `/v1/webhooks/payment-provider` | Card payment provider | HMAC signature header | Rejected if the signature fails or the timestamp is more than 5 minutes old |

A request for another business's resource returns 404, not 403, so that resource IDs cannot be probed (NFR-SEC-05).

## 2. Resource and payload conventions

- JSON, camelCase, UTF-8. German text is sent as UTF-8; there is no transliteration.
- IDs are UUIDs. Human-readable references (`orderRef`, `appointmentRef`) are for display and support only.
- **Money** is integer cents with a `Cents` suffix, VAT-inclusive unless the field name says `net`. The API never sends formatted amounts; the app formats "€ 15,70" per locale (NFR-I18N-01).
- **Time** is RFC 3339 with offset (`2026-10-13T09:30:00+02:00`). Dates without time are `YYYY-MM-DD` in Europe/Vienna.
- **Localisation.** `Accept-Language: de-AT` (default) or `en` selects the language of `detail` in errors and of labels such as `openState.label`.
- **Enumerations** are stable English identifiers (`NotCollected`, `REDUCED_10`). Display text comes from the app's translation files.

## 3. Idempotency and concurrency

- Every unsafe POST that creates something or moves money accepts `Idempotency-Key` (UUID). The key is stored for 24 hours with the response, and a repeat returns the stored response. Required on: `createSlotHold`, `placeOrder`, `cancelBooking`, `bookAppointment`, `transitionOrder`, `refundOrder`.
- A key reused with a different body returns 422 `IDEMPOTENCY_KEY_REUSED`.
- Capacity decisions are made in the database, not in the request handler ([ADR-002](../03-design/architecture/adr/ADR-002-slot-holds-redis-database-guard.md)). Clients must handle 409 with `alternatives`.
- Updates to configuration resources (`PUT`) use `If-Match` with the resource ETag; a stale ETag returns 412 `PRECONDITION_FAILED`.

## 4. Pagination, filtering and sorting

Cursor pagination with `cursor` and `limit` (1 to 50, default 20). Lists return `nextCursor`, which is null on the last page. Sorting is fixed per endpoint and stated in the description. For example, `listLocations` sorts by distance, or by name when no point is given (BR-006). The client cannot request a different ranking.

## 5. Caching

| Resource | Cache-Control | Why |
|---|---|---|
| Images on the CDN | `public, max-age=604800, immutable` (content-addressed URLs) | Never change in place |
| `getLocation` | `public, max-age=60` | Opening state may change |
| `getCatalog` | `max-age=30, must-revalidate`, with an ETag per declaration and price state | NFR-DAT-02. Before CR-005 this was `max-age=86400` at the CDN, which caused INC-2026-006 |
| `listPickupSlots`, `getAvailability` | `no-store` | Capacity changes by the second |
| Everything under `/business` and `/platform` | `no-store` | Private and live |

## 6. Errors

Errors use RFC 9457 problem details (`application/problem+json`) with a stable `code`. `detail` is localized. Validation errors list each field in `errors[]`. Apps show `detail` or their own translation of `code`; they never show the raw code (NFR-USE-02).

| Code | HTTP | Meaning | Message (de-AT) | Rule |
|---|---|---|---|---|
| VALIDATION_FAILED | 422 | One or more fields invalid | Field messages as in SRS section 3.2 | NFR-USE-02 |
| UNAUTHENTICATED | 401 | Missing or expired token | Bitte melde dich erneut an. | NFR-SEC-03 |
| FORBIDDEN_FOR_ROLE | 403 | Role may not call this operation | Diese Funktion ist für Ihre Rolle nicht verfügbar. | BR-039 |
| NOT_FOUND | 404 | Resource does not exist or belongs to another business | Nicht gefunden. | NFR-SEC-05 |
| INVALID_CREDENTIALS | 401 | Unknown email or wrong password | E-Mail-Adresse oder Passwort ist falsch. | BR-004 |
| ACCOUNT_LOCKED | 423 | Locked after 5 failures | Zu viele Versuche. Bitte versuche es in 15 Minuten erneut. | BR-004 |
| EMAIL_IN_USE / MOBILE_IN_USE | 409 | Uniqueness | See SRS 3.2.1 | BR-002 |
| EMAIL_RESERVED_FOR_PORTAL | 409 | Email belongs to a portal account | Diese E-Mail-Adresse kann in der App nicht verwendet werden. | BR-002 |
| CODE_INVALID / CODE_EXPIRED / CODE_ATTEMPTS_EXHAUSTED | 400 | Password reset code problems | See SRS 3.2.1 | BR-004 |
| RESEND_TOO_SOON | 429 | Code requested within 60 s | Neuer Code in 0:{ss}. | BR-004 |
| OPEN_BOOKINGS_EXIST | 409 | Deletion while an order or appointment is open | Du hast noch offene Bestellungen oder Termine. | BR-005 |
| SLOT_FULL | 409 | Slot full when holding | Diese Abholzeit ist gerade ausgebucht. | BR-018 |
| SLOT_FULL_AT_AUTHORISATION | 409 | Slot filled during payment; `alternatives` included | Deine Abholzeit ist inzwischen ausgebucht. Wähle eine neue Zeit. | BR-020 |
| ITEM_CHANGED | 409 | Price or allergen version changed; `errors[]` per line | Die Angaben für „{Artikel}“ haben sich geändert. | BR-013 |
| ITEM_SOLD_OUT | 409 | A line is sold out | „{Artikel}“ ist ausverkauft. | BR-014 |
| LOCATION_NOT_ACCEPTING_ORDERS | 422 | Paused, closed or not Live | {Geschäft} nimmt gerade keine Bestellungen an. | BR-017, BR-040 |
| ORDER_LIMIT_EXCEEDED | 422 | In-app total above EUR 400.00 | In der App kannst du Bestellungen bis € 400,00 bezahlen. | BR-038 |
| VOUCHER_UNKNOWN / VOUCHER_NOT_VALID_NOW / VOUCHER_EXHAUSTED / VOUCHER_OTHER_BUSINESS / VOUCHER_MIN_ORDER | 422 | Voucher table 3.4a rows G1 to G5 | See business rules table 3.4a | BR-036 |
| CANCELLATION_CUTOFF_PASSED | 409 | Too late to cancel or reschedule in the app; `businessPhone` included | Bitte ruf {Geschäft} an: {Telefon}. | BR-023, BR-029 |
| TIME_NO_LONGER_FREE | 409 | Appointment time taken; `alternatives` included | Dieser Termin ist leider nicht mehr frei. | BR-028 |
| INVALID_TRANSITION | 409 | Status change not in the state machine | Diese Aktion ist im aktuellen Status nicht möglich. | BR-021, BR-030 |
| ACCEPT_DEADLINE_PASSED | 409 | Order already auto-rejected | Die Bestellung wurde bereits automatisch abgelehnt. | BR-022 |
| REASON_REQUIRED | 422 | Reject, cancel or refund without a reason | Bitte wählen Sie einen Grund. | BR-022, BR-031, BR-034 |
| PICKUP_CODE_MISMATCH | 422 | Wrong code at hand-over | Code stimmt nicht. Bitte noch einmal prüfen. | BR-024 |
| HANDOVER_LOCKED | 422 | 5 wrong codes in 10 minutes | Übergabe für 5 Minuten gesperrt. | BR-024 |
| PAID_AT_COUNTER_REQUIRED | 422 | Pay-at-pickup hand-over without confirmation | Bitte bestätigen Sie die Zahlung an der Kassa. | BR-024 |
| NOT_COLLECTED_TOO_EARLY | 409 | Before slot end plus 60 minutes | Erst ab {HH:mm} möglich. | BR-024 |
| NO_SHOW_TOO_EARLY | 409 | Before start plus 15 minutes | Erst ab {HH:mm} möglich. | BR-030 |
| CAPTURE_FAILED | 409 | Issuer refused capture at acceptance | Zahlung fehlgeschlagen. Die Bestellung wurde storniert. | BR-033 |
| REFUND_EXCEEDS_CAPTURED | 422 | Refund above the remaining captured amount | Höchstens € {Betrag} erstattbar. | BR-034 |
| DECLARATION_INCOMPLETE / CHOICE_DECLARATION_MISSING / CONFIRMATION_REQUIRED | 422 | Allergen publication gate | See SRS 3.2.6 | BR-011, BR-013 |
| APPOINTMENTS_IN_CONFLICT | 409 | Working-hours change overlaps appointments | {n} Termine sind betroffen. Bitte umbuchen oder absagen. | BR-031 |
| CLOSURE_HAS_PREORDERS | 409 | Closure overlaps accepted pre-orders | {n} Vorbestellungen sind betroffen. | FR-CAT-02 |
| GO_LIVE_GATE_FAILED | 409 | Publish while table 3.8 fails; failing rows listed | Noch nicht alle Voraussetzungen erfüllt. | BR-043 |
| UID_CHECK_PENDING | 409 | Approve before the UID number is confirmed | Die UID-Prüfung ist noch offen. | BR-042 |
| OUTSIDE_SERVICE_AREA | 422 | Location outside Vienna | Quartier ist derzeit nur in Wien verfügbar. | FR-ONB-01 |
| IDEMPOTENCY_KEY_REUSED | 422 | Same key, different body | n/a (developer error) | Section 3 |
| RATE_LIMITED | 429 | Too many requests | Bitte versuche es gleich noch einmal. | NFR-SEC-01 |

## 7. Rate limits

| Operation | Limit | Rule |
|---|---|---|
| `signIn` | 10 per minute per IP and device; lock per account after 5 failures | BR-004 |
| `requestResetCode` | 1 per 60 s per email; 5 per hour per IP | BR-004 |
| `quoteCheckout` with a voucher | 10 voucher attempts per 10 minutes per customer | BR-036 |
| Public reads | 600 per minute per IP | NFR-CAP-01 |

## 8. Versioning and deprecation

The URL carries the major version (`/v1`). Additive changes (new optional fields, new enum values that clients must treat as unknown) do not change the version. The app sends `App-Version`; the API can return 426 `APP_UPDATE_REQUIRED` for builds below the minimum supported version. This was used after INC-2026-006 to retire builds that cached menus too long.

## 9. Configuration endpoints

These portal endpoints follow the same conventions. They are documented in the internal API reference, not in the public contract, because they are simple create, read, update and delete operations on configuration data. They are listed here for traceability.

| Endpoint | Used by | Requirements |
|---|---|---|
| `GET/PUT /v1/business/profile` | Business Admin: name, description, logo, cover, gallery, brand color (contrast checked), legal information | FR-CAT-01 |
| `GET/POST/DELETE /v1/business/users` | Business Admin: invite, change role, deactivate (ends sessions) | FR-CAT-07 |
| `GET/POST/PUT /v1/business/vouchers` | Business Admin: business-funded voucher codes | FR-CAT-09 |
| `GET/PUT /v1/platform/tax-rates` | Platform Admin: effective-dated VAT rates per tax category | FR-ONB-06 |
| `GET /v1/platform/audit-events` | Platform Admin: search by business, actor and date | FR-ONB-07 |
| `GET /v1/platform/feedback`; `POST /v1/me/bookings/{bookingId}/feedback` | Platform Admin reads; customer sends private feedback | FR-ONB-08 |
| `GET/POST/PUT /v1/platform/vouchers` | Platform Admin: Quartier-funded vouchers | FR-ONB-09 |
| `GET /v1/platform/customers/{customerId}/export` | Platform Admin: JSON export for an access request | FR-ACC-09 |

## 10. Webhooks from the card payment provider

- The signature and timestamp are verified before any parsing of business content.
- The provider event ID is inserted into `webhook_events` with a unique constraint. A duplicate returns 200 and is ignored (BR-033, SD-4).
- Processing happens in the worker, not in the request. The endpoint answers within 2 seconds so the provider does not retry unnecessarily.
- Hourly reconciliation queries the provider for payments that are not in a final state 15 minutes after their last event (FR-PAY-09).

## Related documents

- [OpenAPI specification](openapi.yaml)
- [Realtime events](realtime-events.md)
- [Sequence diagrams](../03-design/diagrams/sequence-diagrams.md)
- [Software Requirements Specification](../02-requirements/SRS.md)
- [Business rules](../02-requirements/business-rules.md)
