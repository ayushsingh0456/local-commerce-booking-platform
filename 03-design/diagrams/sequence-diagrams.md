# Sequence Diagrams: Quartier

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DES-SEQ |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst (co-authored with the Engineering Lead) |
| Last updated | 2026-09-25 |
| Reviewers | Engineering Lead; QA Lead; Finance Lead |

**Purpose and scope.** The runtime interactions behind the rules that involve time, concurrency or money. Each diagram names the rules it implements so QA can test the same path. Participant names match the containers in the [system architecture](../architecture/system-architecture.md).

## SD-1 Checkout with slot hold, 3-D Secure and re-check (R1.1)

Implements BR-019, BR-020, BR-033, FR-ORD-03, FR-ORD-12 and FR-PAY-01. The timings are those of the example in business rules table 3.3.

```mermaid
sequenceDiagram
  autonumber
  participant App as Customer App
  participant API as Quartier API
  participant R as Redis
  participant DB as PostgreSQL
  participant PSP as Card payment provider
  App->>API: POST /slot-holds (location, slot 12:15)
  API->>R: SET hold key, TTL 600 s
  API-->>App: 201 hold, expires 11:50:00
  App->>API: POST /checkout-quotes (lines with declaration versions, voucher)
  API-->>App: 200 totals, VAT per rate, changes: none
  App->>API: POST /orders (Idempotency-Key, holdId, payment method)
  API->>DB: Re-check prices and allergen versions
  API->>PSP: Create payment (authorize only, manual capture)
  PSP-->>App: 3-D Secure challenge in banking app
  PSP-->>API: Webhook payment.challenge_started at 11:43:10
  API->>R: Extend hold once by 300 s (expires 11:55:00)
  Note over App,PSP: Customer approves in the banking app at 11:56:20
  PSP-->>API: Webhook payment.authorized
  API->>DB: BEGIN, lock pickup_slot_counters row (12:15)
  alt capacity free (own hold active or a unit free)
    API->>DB: INSERT order Placed, increment counter, COMMIT
    API-->>App: Order Placed, pickup code K7PM
  else slot full
    API->>DB: ROLLBACK
    API->>R: Hold next 3 free slots for 180 s
    API-->>App: 409 SLOT_FULL_AT_AUTHORISATION, alternatives 12:30, 12:45, 13:00
    App->>API: POST /orders (same paymentRef, holdId of 12:30) within 3 min
    API->>DB: Lock 12:30 counter, INSERT order Placed, COMMIT
  end
```

If no alternative is chosen in 3 minutes, the API voids the authorization and the app shows "Reservierung aufgehoben, es wurde nichts abgebucht".

## SD-2 Order board: accept, capture and live status

Implements BR-021, BR-022, BR-033, FR-POS-01, FR-POS-02 and FR-ORD-06.

```mermaid
sequenceDiagram
  autonumber
  participant API as Quartier API
  participant WS as Realtime gateway
  participant POS as POS view (tablet)
  participant PSP as Card payment provider
  participant W as Worker
  participant App as Customer App
  API->>WS: order.placed (location L-1001)
  WS-->>POS: order.placed, accept deadline 12:01:30
  POS->>POS: Sound and flash until acted on
  W->>W: Timer for accept deadline
  POS->>API: POST /business/orders/{id}/transitions {accept}
  API->>PSP: Capture amount, application fee 1.51
  PSP-->>API: Captured
  API->>WS: order.updated Accepted
  WS-->>App: status Accepted
  API->>W: Cancel accept timer
  Note over W,App: Without acceptance by the deadline the worker auto-rejects, voids and notifies
  POS->>API: transitions {ready}
  API->>WS: order.updated Ready
  WS-->>App: status Ready
  API->>App: Push "Deine Bestellung ist fertig"
```

## SD-3 Appointment hold and confirmation with a database conflict

Implements BR-026 to BR-028, FR-APT-03 and FR-APT-04. Two customers try to book Lena Auer at 09:30 on 2026-10-13.

```mermaid
sequenceDiagram
  autonumber
  participant A as Customer A (Sophie)
  participant B as Customer B
  participant API as Quartier API
  participant R as Redis
  participant DB as PostgreSQL
  A->>API: GET /locations/L-2001/availability (service, Lena, 2026-10-13)
  API->>DB: Working intervals, appointments, absences
  API->>R: Active holds for Lena
  API-->>A: 09:00, 09:15, 09:30, 11:15, 11:30, 11:45, 12:00
  A->>API: POST /slot-holds (staff Lena, 09:30 to 10:30)
  API->>R: SET hold, TTL 600 s
  B->>API: GET availability (same day)
  API-->>B: 11:15, 11:30, 11:45, 12:00 (09:00 to 09:30 now overlap the hold)
  Note over B,API: B's app had cached 09:30 from an earlier view
  B->>API: POST /appointments (09:30, no hold)
  A->>API: POST /appointments (09:30, holdId)
  API->>DB: INSERT appointment B, range 09:30 to 10:30
  API->>DB: INSERT appointment A, range 09:30 to 10:30
  DB-->>API: B committed first
  DB-->>API: A rejected by exclusion constraint
  API-->>A: 409 TIME_NO_LONGER_FREE, nearest: 11:15, 11:30, 11:45
```

The hold did not stop customer B's request, because B sent no hold. The constraint did. In production, a request without a hold that overlaps someone else's active hold is rejected before the insert (table 3.5 check E6). The diagram shows the last line of defense.

## SD-4 Card payment provider webhook, processed once

Implements BR-033, FR-PAY-09 and FR-ONB-03.

```mermaid
sequenceDiagram
  autonumber
  participant PSP as Card payment provider
  participant API as Quartier API
  participant DB as PostgreSQL
  participant W as Worker
  PSP->>API: POST /webhooks/payment-provider (event evt_8812, signature)
  API->>API: Verify signature and timestamp (max 5 min skew)
  API->>DB: INSERT webhook_events (provider_event_id evt_8812)
  alt first delivery
    DB-->>API: Inserted
    API->>W: Enqueue process evt_8812
    API-->>PSP: 200
    W->>DB: Apply state change (for example Authorised to Captured)
  else repeated delivery
    DB-->>API: Unique violation
    API-->>PSP: 200 (already received)
  end
  Note over W,DB: Hourly reconciliation compares non-final payments older than 15 min with the provider
```

## SD-5 Allergen declaration published and seen by the customer (after CR-005)

Implements BR-011, BR-013, FR-CAT-04, FR-ORD-12 and NFR-DAT-02.

```mermaid
sequenceDiagram
  autonumber
  participant Adm as Business Admin (portal)
  participant API as Quartier API
  participant DB as PostgreSQL
  participant App as Customer App
  Adm->>API: POST /business/items/{id}/allergen-declarations {A,C,G,H, confirmed}
  API->>DB: INSERT declaration version 4, UPDATE item current version
  API->>DB: INSERT audit_event allergen.confirmed
  API-->>Adm: 201 version 4, live within 60 s
  App->>API: GET /locations/L-1002/catalog (If-None-Match: v3 ETag)
  API-->>App: 200 new ETag, item shows A C G H
  Note over App: Cart line still stores version 3
  App->>API: POST /checkout-quotes (line version 3)
  API-->>App: 200 changes: allergens added H for line 1
  App->>App: Line expanded, H in bold, confirm required
  App->>API: POST /orders (line version 4 confirmed)
```

## SD-6 Business payment onboarding (food)

Implements BR-043, FR-ONB-03 and FR-ONB-04.

```mermaid
sequenceDiagram
  autonumber
  participant Adm as Business Admin
  participant BWP as Business Web Portal
  participant API as Quartier API
  participant PSP as Card payment provider
  participant PA as Platform Admin
  Adm->>BWP: Start payment onboarding
  BWP->>API: Request onboarding link
  API->>PSP: Create connected account, request hosted onboarding link
  PSP-->>API: Link (valid 24 h)
  API-->>BWP: Redirect to hosted onboarding
  Adm->>PSP: Identity, owners, IBAN
  PSP-->>API: Webhook account.verification_updated: Verified
  API->>API: Go-live gate L2 passes
  PA->>API: POST /platform/businesses/{id}/status-changes {publish}
  API-->>PA: 200 Live (all gate rows pass)
```

## Related documents

- [Process flows](process-flows.md)
- [State machines](state-machines.md)
- [ADR-002 Slot holds](../architecture/adr/ADR-002-slot-holds-redis-database-guard.md)
- [ADR-003 Marketplace payments](../architecture/adr/ADR-003-marketplace-payments-authorize-capture.md)
- [Realtime events](../../04-api/realtime-events.md)
- [OpenAPI specification](../../04-api/openapi.yaml)
