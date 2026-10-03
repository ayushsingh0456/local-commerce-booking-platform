# ADR-002: Slot holds in Redis, the final decision in PostgreSQL

## Document control

| Field | Value |
|---|---|
| Status | Accepted (v1.1 amended by CR-006) |
| Date | 2026-03-04 (v1.0); 2026-09-14 (v1.1) |
| Deciders | Engineering Lead, Product Owner, Business Analyst |
| Consulted | Backend developers, QA Lead, Finance Lead (v1.1) |
| Related requirements | BR-018, BR-019, BR-020, BR-026, BR-028, FR-ORD-03, FR-ORD-12, FR-APT-03, FR-APT-04, NFR-PERF-03 |
| Supersedes | None |

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-03-04 | Holds in Redis with TTL; database constraints guard confirmation |
| 1.1 | 2026-09-14 | After INC-2026-011: the hold is not a guarantee at payment time; capacity is re-checked at authorization; one 5-minute extension when 3-D Secure starts (CR-006) |

## Context and problem statement

Two kinds of capacity are sold in Quartier:
- **Pickup slots.** A count: at most N orders per 15-minute slot per location (BR-018).
- **Staff time.** An interval: a stylist cannot be in two appointments, buffers included (BR-026).

Between choosing a slot or start time and confirming, a customer may spend minutes in checkout, and sometimes in a 3-D Secure challenge in their banking app. Another customer must not take the same capacity in that time, and abandoned choices must not block capacity for long.

How do we reserve capacity during checkout without over-selling it and without leaking it?

## Decision drivers

- No over-sold slot and no double-booked staff member (BR-020, BR-028).
- Abandoned checkouts free capacity quickly.
- Availability must compute in 400 ms or less (NFR-PERF-03), which includes subtracting active holds.
- Simple to reason about for QA and for the business ("6 means 6").

## Considered options

1. **Holds as database rows** with an expiry timestamp and a clean-up job.
2. **Holds in Redis only:** the hold is the reservation; confirmation trusts it.
3. **Holds in Redis for the user experience, with the final decision made by the database** under a lock or constraint at confirmation.

## Decision outcome

Chosen option: **3.**
- **Holds.** Each hold is a Redis key with a time-to-live: `hold:pickup:{locationId}:{slotStart}:{holdId}` or `hold:staff:{staffId}:{start}:{end}:{holdId}`. Availability queries subtract active holds. Expiry needs no clean-up job.
- **Pickup confirmation.** The order is inserted in the same transaction that locks the slot's counter row (`SELECT ... FOR UPDATE` on `pickup_slot_counters`) and checks the count of active orders against capacity.
- **Appointment confirmation.** An exclusion constraint on `appointments` prevents overlapping `tstzrange(starts_at, occupied_until)` for the same `staff_id` among Confirmed and Checked-in rows. The database rejects a double booking even if two requests pass the hold check at the same moment.

**v1.1 amendment (CR-006).** Version 1.0 treated an active hold as permission to create the pickup order without re-checking the count. INC-2026-011 showed the gap. During a slow issuer, 3-D Secure challenges outlasted the 10-minute holds, other customers took the freed capacity, and the delayed payments then created orders in full slots. From v1.1:
- When the card payment provider reports a challenge start, the hold is extended once by 5 minutes (BR-019).
- The order is created only through the counter-row check, whether or not the hold is still active (BR-020).
- If the slot is full at authorization, the customer is offered the next 3 free slots, held for 3 minutes. The authorization is kept, and voided if the customer does not choose.

### Positive consequences

- Holds expire by themselves; no job to fail.
- Over-selling is impossible by construction. A hold is a courtesy to the customer, not a guarantee to the system.
- Appointments are protected by a declarative constraint that QA can test with two concurrent requests (TC-APT-005).

### Negative consequences

- A customer can lose a slot they saw as held if the hold expired during payment. v1.1 reduces this with the extension and alternative slots, and the copy is honest about it.
- Redis is now on the path of availability queries. If Redis is unavailable, holds are not created. Checkout then proceeds without a hold, and the database check still protects capacity.

## Pros and cons of the other options

| Option | Pros | Cons |
|---|---|---|
| 1 Rows with expiry | One store; transactional | Clean-up job on the critical path; expired rows block capacity until it runs; heavier availability queries |
| 2 Redis only | Fast, simple | A hold that expires during payment and a new hold for the last unit can both convert. This is exactly the INC-2026-011 failure mode |

## Validation

- TC-ORD-003: 5 orders plus 1 hold make the slot full.
- TC-ORD-005: a hold expires at 10 minutes.
- TC-ORD-010: a slow 3-D Secure challenge on a full slot leads to alternatives, never an order in the full slot.
- TC-APT-005: two concurrent confirmations for the same staff interval give one success and one conflict with alternatives.
- NFR-OBS-02 alerts if any slot is ever over capacity.

## Links

- [Sequence diagrams: checkout with hold and 3-D Secure](../../diagrams/sequence-diagrams.md)
- [INC-2026-011 Pickup slots over capacity](../../../07-operations/incidents/INC-2026-011-pickup-slots-over-capacity.md)
- [Business rules table 3.3](../../../02-requirements/business-rules.md)

## Related documents

- [System architecture](../system-architecture.md)
- [Change request log (CR-006)](../../../05-delivery/change-request-log.md)
