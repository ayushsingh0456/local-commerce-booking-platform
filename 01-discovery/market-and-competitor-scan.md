# Market and Competitor Scan: Quartier

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DIS-MKT |
| Version | 1.0 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-02-20 |
| Reviewers | Product Owner; Managing Director; Marketing Lead |

**Purpose and scope.** The alternatives that Vienna's independent businesses and their customers use today, grouped by archetype, and where Quartier positions itself. Competitors are described as archetypes rather than named products. The comparison is about patterns that matter for requirements, and individual products change too often for a named comparison to stay accurate. Fee ranges are indicative public price ranges collected in January 2026 and rounded.

## 1. Alternatives by archetype

| Archetype | What it does | Typical cost to the business | Strengths | Weaknesses for our pilot businesses |
|---|---|---|---|---|
| A. Pan-European delivery marketplace | Food ordering with delivery and pickup; ranking and promoted listings | 15% to 30% commission; pickup often 8% to 15% | Huge reach; brand recognition | High fees; paid ranking; businesses feel anonymous; allergen detail varies by listing |
| B. Salon booking marketplace | Online booking for salons and spas, often with a marketplace of new customers | Monthly subscription plus a fee for new customers acquired through the marketplace | Mature booking engines | Salon-only; ratings pressure; separate from food |
| C. Booking widget or software for a single business | Diary software with a booking page | Monthly subscription | Good diaries; the business owns the customer | No discovery; customers juggle many apps and pages |
| D. Cash register vendor with an online ordering add-on | Online ordering attached to the business's register | Add-on subscription; card fees | Integrated with the register | Per-vendor; no shared customer app; little discovery |
| E. Phone, social media and messenger | Orders and bookings by phone, direct message or messenger | Free | No fees; personal | Interrupts work; no capacity control; no prepayment; allergen questions repeated every time |

## 2. Customer research highlights

From the survey of 410 residents of Neubau and Mariahilf (January 2026):
- **Channel.** 72% had ordered lunch for pickup at least once a month. 41% would rather order in an app than call. 23% had given up calling because the line was engaged.
- **Allergies.** 18% reported a food allergy or intolerance in their household. Of those, 64% said they ask staff about allergens "every time".
- **Ratings.** 52% said star ratings influence their choice of a new place. Among respondents choosing among places they already knew, only 14% did. This informed DEC-05 and later CR-007, together with the business survey: 11 of 14 owners were against public ratings.

## 3. Positioning

| Dimension | Quartier R1 | Rationale |
|---|---|---|
| Categories | Food, salon and spa in one app | Local life in a district does not divide into apps; discovery across categories |
| Ranking | Distance or name only; no promoted placement, no public ratings | Fairness for small businesses; transparency under P2B rules (BR-006, BR-008) |
| Fees | 8% on in-app paid food orders; appointments free in R1 | Clearly below archetype A; no fee where Quartier does not handle money |
| Fulfillment | Pickup only | Avoids courier logistics; matches the lunch use case |
| Allergens | Mandatory, versioned, current | A differentiator for 18% of households, and a legal requirement for distance sales |
| Business identity | Logo, cover and brand color on the business page | Owners want their own identity, not a uniform listing |
| Onboarding | Self-service, live in days | Archetypes B and C take one to three weeks to set up |

## 4. Implications for requirements

| Finding | Requirement response |
|---|---|
| Delivery platforms rank by fee and promotion | BR-008 neutral ranking; FR-DSC-04 cards without badges |
| Allergen quality varies across platforms | BR-011 publishing gate; BR-013 versions |
| Salons fear ratings more than fees | No public ratings (DEC-05); private feedback only (FR-ONB-08) |
| Set-up time loses businesses | Go-live gate as a checklist (BR-043); OBJ-05 |
| Register vendors already cover counter payments | POS view without cash functions (DEC-11, CR-003) |

## Related documents

- [Project charter](project-charter.md)
- [Current vs future state](current-vs-future-state.md)
- [Business Requirements Document](../02-requirements/BRD.md)
- [Decision log](../05-delivery/decision-log.md)
