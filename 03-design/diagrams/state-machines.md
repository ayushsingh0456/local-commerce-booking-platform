# State Machines: Quartier

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DES-SM |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Engineering Lead; QA Lead; Partner Operations Lead |

**Purpose and scope.** The lifecycles of the six entities whose status drives behavior. Every transition names its trigger and the rule that allows it. A transition that is not drawn is not allowed, and the API returns `409 INVALID_TRANSITION` for it. QA derives state-transition test cases from these diagrams.

## SM-1 Order status (BR-021 to BR-024)

```mermaid
stateDiagram-v2
  state "Not collected" as NotCollected
  [*] --> Placed: order created (BR-020)
  Placed --> Accepted: staff accept, payment captured
  Placed --> Rejected: staff reject with reason
  Placed --> Rejected: no answer in 5 min (auto)
  Placed --> Canceled: customer cancels, authorization voided
  Accepted --> Preparing: staff start
  Accepted --> Canceled: customer cancels before cutoff, full refund
  Preparing --> Ready: staff mark ready
  Ready --> Completed: pickup code matched
  Ready --> NotCollected: from slot end + 60 min
  Completed --> [*]
  Rejected --> [*]
  Canceled --> [*]
  NotCollected --> [*]
```

A delay (+10, +15, +30 minutes) does not change the status; it changes the ready time and notifies the customer. The app never moves an order forward; only staff actions and the acceptance timeout do.

## SM-2 Payment state (BR-032 to BR-034, table 3.7)

```mermaid
stateDiagram-v2
  state "Due at pickup" as DueAtPickup
  state "Paid at counter" as PaidAtCounter
  state "Partially refunded" as PartiallyRefunded
  [*] --> Authorised: in-app order placed
  [*] --> DueAtPickup: pay-at-pickup order placed
  Authorised --> Captured: business accepts
  Authorised --> Voided: rejected, auto-rejected or canceled
  Authorised --> Failed: capture fails (K8)
  Captured --> PartiallyRefunded: business refunds part
  Captured --> Refunded: full refund
  PartiallyRefunded --> PartiallyRefunded: further partial refund
  PartiallyRefunded --> Refunded: refunds reach captured amount
  DueAtPickup --> PaidAtCounter: staff confirm at hand-over
  Voided --> [*]
  Refunded --> [*]
  Failed --> [*]
  PaidAtCounter --> [*]
```

## SM-3 Appointment status (BR-029 to BR-031)

```mermaid
stateDiagram-v2
  state "Checked in" as CheckedIn
  state "No-show" as NoShow
  [*] --> Confirmed: booked (constraint passed)
  Confirmed --> CheckedIn: staff check in
  CheckedIn --> Completed: staff complete
  Confirmed --> Canceled: customer before cutoff
  Confirmed --> Canceled: business with reason
  Confirmed --> Canceled: replaced by a reschedule
  Confirmed --> NoShow: staff, from start + 15 min
  Completed --> [*]
  Canceled --> [*]
  NoShow --> [*]
```

## SM-4 Business lifecycle (BR-042 to BR-044)

```mermaid
stateDiagram-v2
  state "In review" as InReview
  state "Info requested" as InfoRequested
  [*] --> Applied: application submitted
  Applied --> InReview: Platform Admin opens it
  InReview --> InfoRequested: more information needed
  InfoRequested --> InReview: applicant answers
  InReview --> Rejected: reject with reason
  InReview --> Approved: approve
  Approved --> Live: publish when the go-live gate passes
  Live --> Suspended: suspend with statement of reasons
  Suspended --> Live: reinstate
  Live --> Closed: contract ended with 30 days notice
  Suspended --> Closed: contract ended
  Rejected --> [*]
  Closed --> [*]
```

## SM-5 Item publication and allergen declaration (BR-011, BR-013)

```mermaid
stateDiagram-v2
  state "Change pending" as ChangePending
  [*] --> Draft: item created
  Draft --> Published: declaration complete and confirmed (version 1)
  Published --> ChangePending: ingredients or allergens edited
  ChangePending --> Published: confirmed, version n + 1 live within 60 s
  ChangePending --> Published: edit discarded, version n stays live
  Published --> Hidden: hidden from the app
  Hidden --> Published: shown again
  Draft --> [*]: deleted
  Hidden --> [*]: deleted
```

While an item is in Change pending, customers keep seeing the last confirmed version. Unconfirmed edits are never shown in the app.

## SM-6 Hold (BR-019, BR-028)

```mermaid
stateDiagram-v2
  state "Active (extended)" as Extended
  [*] --> Active: slot or start time chosen, TTL 10 min
  Active --> Extended: 3-D Secure challenge started, +5 min once
  Active --> Converted: order or appointment created
  Extended --> Converted: order created
  Active --> Released: customer changes choice or leaves
  Active --> Expired: TTL reached
  Extended --> Expired: TTL reached
  Converted --> [*]
  Released --> [*]
  Expired --> [*]
```

An Expired hold does not stop the order. The re-check at authorization decides (BR-020, SD-1).

## Related documents

- [Process flows](process-flows.md)
- [Sequence diagrams](sequence-diagrams.md)
- [Business rules](../../02-requirements/business-rules.md)
- [Data dictionary](../data/data-dictionary.md)
- [Test cases](../../06-quality/test-cases.md)
