# ADR-003: Marketplace payments through connected accounts, authorize then capture at acceptance

## Document control

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-02-25 |
| Deciders | Managing Director, Engineering Lead, Product Owner, Finance Lead, Business Analyst |
| Consulted | External legal counsel, card payment provider solutions team, Compliance Lead |
| Related requirements | BR-032 to BR-038, FR-PAY-01 to FR-PAY-09, FR-ONB-03, NFR-SEC-02 |
| Supersedes | None |

## Context and problem statement

Customers pay for food orders in the app. The money belongs to the business, which is the seller. The platform earns a commission. Customers expect to be charged only for what they get. Businesses reject orders when they are too busy or an item runs out (BR-022).

Two questions follow:
1. Who receives the customer's money?
2. When is the card charged?

## Decision drivers

- The platform must not hold customer funds. That would require a payment institution license (compliance mapping P3).
- No card data in the platform systems (NFR-SEC-02).
- A rejected order should cost nobody anything: no refund, no fee, no wait for money back.
- Commission collected automatically, with VAT, and reconcilable weekly (BR-037).
- Strong customer authentication must work, including long challenges (BR-032).

## Considered options

**Money flow**
1. The platform as merchant of record: collects all payments and pays businesses out.
2. Each business has its own merchant account with any provider; the platform invoices commission monthly.
3. **Marketplace with connected accounts at one card payment provider**: the payment settles into the business's connected account; the platform's commission is taken as an application fee.

**Charge timing**
- A. Capture at placement.
- B. **Authorise at placement, capture at acceptance.**
- C. Authorise at placement, capture at handover.

## Decision outcome

Chosen: **3 with B.**
- **Onboarding.** Each food business completes the provider's hosted onboarding (identity, beneficial owners, IBAN) before go-live (BR-043, L2). The provider performs the checks.
- **Placement.** The order total is authorized with 3-D Secure where the issuer asks for it. Nothing is charged yet.
- **Acceptance.** The authorization is captured, with the commission (8% plus 20% VAT) as the application fee (BR-037).
- **Rejection, auto-rejection or customer cancellation before acceptance.** The authorization is voided. The customer sees "Reservierung aufgehoben, es wurde nichts abgebucht".
- **Cancellation after acceptance, within the cutoff.** A full refund (BR-023); the provider reverses the application fee pro rata.
- **Payouts.** The provider pays the business weekly. The platform shows statements built from provider data (FR-PAY-08).

### Positive consequences

- The platform never holds funds and never sees card numbers.
- Rejections, the most common exception in the pilot, cost nothing and need no refund.
- Commission and VAT on commission are computed once, at capture, and reconcile with the provider's payout report.

### Negative consequences

- Authorisations can expire. Same-day orders are captured within minutes. Pre-orders up to 7 days ahead are accepted within 30 minutes of the next opening (BR-022), so they are captured long before the authorization would lapse.
- A capture can fail when the issuer has canceled the authorization. Table 3.7 row K8 handles this: the order is canceled, both sides are told, and an alert is raised.
- Appointments are paid at the venue in R1, so the payment set-up is not needed for salons and spas. This makes their onboarding shorter (BR-043).

## Pros and cons of the other options

| Option | Pros | Cons |
|---|---|---|
| 1 Merchant of record | Full control of the money flow | Needs a payment license or a licensed partner holding funds; the platform becomes the seller for VAT purposes unless carefully structured |
| 2 Business's own accounts | No marketplace contract | Commission collected by monthly invoice and dunning; every business integrates a different provider; no single checkout experience |
| A Capture at placement | Simplest | Every rejection becomes a refund that takes days to reach the customer |
| C Capture at handover | Charges only for food collected | Business carries the risk of uncollected prepared food, which undermines OBJ-01; long authorization holds for pre-orders |

## Validation

- TC-PAY-002: authorize, capture at accept, void at reject.
- TC-PAY-005: refunds and the cap at the captured amount.
- TC-PAY-007: webhook idempotency.
- TC-PAY-008: commission and payout for the canonical basket.
- Weekly reconciliation: orders captured, minus refunds, minus commission, equals payouts. Run by the Finance Lead since the pilot, with no unexplained difference.

## Links

- [Business rules table 3.7](../../../02-requirements/business-rules.md)
- [Sequence diagrams](../../diagrams/sequence-diagrams.md)
- [Compliance mapping](../../../02-requirements/compliance-mapping.md)

## Related documents

- [System architecture](../system-architecture.md)
- [State machines](../../diagrams/state-machines.md)
