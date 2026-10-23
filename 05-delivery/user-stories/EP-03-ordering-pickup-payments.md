# EP-03 Ordering, Pickup & Payments: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | LC-DEL-US-03 |
| Version | 1.3 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Product Owner, Engineering Lead, QA Lead, Finance Lead, Compliance Lead |

## Purpose and scope

User stories for EP-03:
- cart;
- pickup slots and holds;
- payment with 3-D Secure;
- VAT-inclusive totals with vouchers;
- order status and pickup;
- cancellation and refunds;
- rebook and pre-orders;
- receipts;
- the R1.1 re-check before payment.

Version 1.1 added US-018 (CR-001). Version 1.3 adds US-020 (CR-005 and CR-006, from INC-2026-006 and INC-2026-011).

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-03 |
| Name | Ordering, Pickup & Payments |
| Modules | ORD (Ordering & Pickup), PAY (Payments & Receipts) |
| Goal | Let a customer order food for pickup in a slot the kitchen can handle, pay securely with bank confirmation, see exactly what VAT they paid, and collect without waiting at the counter. |
| Objectives | OBJ-01 Cut uncollected pickup orders; OBJ-06 Free counter staff at lunch |
| Business need | BN-03 |
| Functional requirements | FR-ORD-01 to FR-PAY-09 (21) |
| Stories | US-011 to US-020 (10, 48 points) |
| Release | R1; US-020 in R1.1 |

## Story list

| Story | Title | Persona | Priority | Points | Sprint | Release |
|---|---|---|---|---|---|---|
| US-011 | Build a cart for one business | PER-01 Sophie Wallner (CUS) | Must | 3 | S3 | R1 |
| US-012 | Choose a pickup slot the kitchen can handle | PER-01 Sophie Wallner (CUS) | Must | 8 | S3 | R1 |
| US-013 | Pay by card or wallet with bank confirmation | PER-01 Sophie Wallner (CUS) | Must | 8 | S4 | R1 |
| US-014 | See a VAT-inclusive total with a voucher | PER-01 Sophie Wallner (CUS) | Must | 5 | S4 | R1 |
| US-015 | Follow my order and collect it | PER-01 Sophie Wallner (CUS) | Must | 5 | S4 | R1 |
| US-016 | Cancel an order and get my money back | PER-01 Sophie Wallner (CUS) | Must | 3 | S5 | R1 |
| US-017 | Rebook a past order | PER-01 Sophie Wallner (CUS) | Should | 3 | S5 | R1 |
| US-018 | Pre-order a whole cake for a later day | PER-02 Daniela Costa (CUS) | Should | 5 | S5 | R1 |
| US-019 | Get a receipt in the business's name | PER-01 Sophie Wallner (CUS) | Must | 3 | S5 | R1 |
| US-020 | Be told what changed before I pay | PER-02 Daniela Costa (CUS) | Must | 5 | S13 | R1.1 |
| **Total** |  |  |  | **48** |  |  |

Shared test data: Café Mohnblume (L-1001), ordering hours 07:30 to 17:30, lead time 20 minutes, slot capacity 6, cancellation cutoff 30 minutes, pay at pickup enabled, voucher MOHN3 (EUR 3.00 off from EUR 15.00, business-funded, once per customer). Konditorei Pomeranze (L-1002), ordering hours 06:30 to 18:30, item "Schokoladentorte, ganz" EUR 38.00 with lead time 48 hours. Sandbox cards of the card payment provider, referred to by their last four digits: card ending 3155 always asks for a 3-D Secure challenge; card ending 0077 is frictionless. Kaffeebar Zimtstern (L-1005) applies the small-business VAT exemption. Dates are Thursday 2026-09-17 unless stated.

## Stories

### US-011 · Build a cart for one business

| Field | Value |
|---|---|
| Epic | EP-03 Ordering, Pickup & Payments |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-ORD-01 |
| Business rules | BR-016 |
| Dependencies | US-010 |
| Test cases | TC-ORD-001 |

**Story**
As a customer, I want a cart that follows me while I browse one business's menu, so that I can collect my order in one place before I pay.

**Acceptance criteria**

```gherkin
Scenario: US-011-AC1 Add items and see the cart bar
  Given Sophie is signed in on the Café Mohnblume page
  When she adds 2 x Topfengolatsche and 1 x Schinken-Käse-Sandwich
  Then the cart bar shows "3 Artikel · € 13,70"

Scenario: US-011-AC2 Adding from another location asks first
  Given Sophie's cart holds items from Café Mohnblume
  When she taps Add on "Pho Bo" at Nudelbar Sesam, Neubau
  Then the dialog reads "Dein Warenkorb enthält Artikel von Café Mohnblume. Neuen Warenkorb mit diesem Artikel starten?"
  When she taps "Aktuellen Warenkorb behalten"
  Then nothing is added and the Café Mohnblume cart is unchanged

Scenario: US-011-AC3 Edit a configured line
  Given the cart has "Hausgemachte Limonade, 0,3 l, Holunder" at € 3,90
  When Sophie opens the line and changes the size to "0,5 l"
  Then the popup button reads "Aktualisieren · € 5,00" and the line shows € 5,00

Scenario Outline: US-011-AC4 Cart expires after 24 hours without activity
  Given Sophie last changed her cart at 12:00 on 2026-09-17
  When she opens the app at <time>
  Then the cart is <state>

  Examples:
    | time              | state     |
    | 2026-09-18 11:59  | kept      |
    | 2026-09-18 12:00  | discarded |
```

### US-012 · Choose a pickup slot the kitchen can handle

| Field | Value |
|---|---|
| Epic | EP-03 Ordering, Pickup & Payments |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-ORD-02, FR-ORD-03 |
| Business rules | BR-017, BR-018, BR-019 |
| Dependencies | US-011 |
| Test cases | TC-ORD-002, TC-ORD-003, TC-ORD-005 |

**Story**
As a customer, I want to choose a pickup time the café can really meet, and have it kept for me while I pay, so that my food is ready when I arrive.

**Acceptance criteria**

```gherkin
Scenario Outline: US-012-AC1 Earliest slot respects the 20-minute lead time
  Given the slot 12:15 has free capacity
  When Sophie opens slot selection at <now>
  Then the first slot offered is "<first slot>"

  Examples:
    | now   | first slot    |
    | 11:52 | 12:15 - 12:30 |
    | 11:55 | 12:15 - 12:30 |
    | 11:56 | 12:30 - 12:45 |

Scenario: US-012-AC2 A full slot is shown but cannot be chosen
  Given slot 12:15 has 5 orders in Placed, Accepted, Preparing or Ready and 1 active hold
  When Sophie opens slot selection at 11:52
  Then 12:15 is shown as "ausgebucht" and cannot be selected
  And the first selectable slot is 12:30 - 12:45

Scenario Outline: US-012-AC3 The hold lasts 10 minutes
  Given Sophie chose 12:15 at 11:40:00 and the checkout shows "Abholzeit reserviert · 10:00"
  When she reaches the order button at <time> without starting payment
  Then the result is "<result>"

  Examples:
    | time     | result                                                                |
    | 11:49:59 | she can place the order                                               |
    | 11:50:00 | "Deine reservierte Abholzeit ist abgelaufen. Bitte wähle sie erneut." |

Scenario: US-012-AC4 Leaving checkout releases the hold
  Given Sophie holds the last unit of slot 12:15
  When she leaves checkout to add another item
  Then her hold is released and another customer can choose 12:15

Scenario: US-012-AC5 No slots after the last ordering slot
  When Sophie opens slot selection at 17:00
  Then she sees "Heute sind keine Abholzeiten mehr frei." and the order button is disabled
```

**Notes**
- Slot logic is business rules table 3.2; the hold mechanism is [ADR-002](../../03-design/architecture/adr/ADR-002-slot-holds-redis-database-guard.md).

### US-013 · Pay by card or wallet with bank confirmation

| Field | Value |
|---|---|
| Epic | EP-03 Ordering, Pickup & Payments |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-PAY-01, FR-PAY-02, FR-PAY-03, FR-PAY-09 |
| Business rules | BR-024, BR-032, BR-033 |
| Dependencies | US-012, US-035 |
| Test cases | TC-PAY-001, TC-PAY-002, TC-PAY-007, TC-PAY-009 |

**Story**
As a customer, I want to pay in the app with my card, Apple Pay or Google Pay and confirm with my bank, so that the order is paid and I am charged only if the café accepts it.

**Acceptance criteria**

```gherkin
Scenario: US-013-AC1 Authorise at placement, capture at acceptance
  Given Sophie places an order of € 15,70 with the sandbox card ending 3155
  When she completes the 3-D Secure challenge
  Then the order is Placed and Booking Details shows "Zahlung reserviert, wird bei Annahme abgebucht"
  When Tobias accepts the order on the order board
  Then € 15,70 is captured with an application fee of € 1,51
  And Booking Details shows "Bezahlt mit Karte •••• 3155"

Scenario: US-013-AC2 Rejection voids the authorization
  Given an order is Placed with an authorized payment
  When Tobias rejects it with the reason "Artikel ausverkauft"
  Then the authorization is voided and nothing is captured
  And Sophie sees "Reservierung aufgehoben, es wurde nichts abgebucht"

Scenario: US-013-AC3 Card data never reaches the platform
  When Sophie enters her card in checkout
  Then the card number is entered in the card payment provider's hosted field
  And no the platform API request, log line or error event contains the card number

Scenario Outline: US-013-AC4 Pay at pickup only where the business offers it
  Given <business> has pay at pickup <setting>
  When Sophie opens the payment methods
  Then "Bezahlung bei Abholung" is <shown>

  Examples:
    | business            | setting  | shown     |
    | Café Mohnblume      | enabled  | shown     |
    | Nudelbar Sesam      | disabled | not shown |

Scenario: US-013-AC5 A declined payment creates no order
  When Sophie's bank declines the 3-D Secure challenge
  Then no order is created
  And she sees "Die Zahlung wurde nicht bestätigt. Es wurde nichts abgebucht."
  And her slot hold remains until it expires

Scenario: US-013-AC6 Webhooks are processed once
  Given the card payment provider sends "payment.authorized" for the order twice with the same event ID
  Then the order is created once and the second delivery is answered 200 and ignored
```

**Notes**
- Payment timing is [ADR-003](../../03-design/architecture/adr/ADR-003-marketplace-payments-authorize-capture.md); the actions per event are table 3.7.
- The application fee for the canonical basket is commission EUR 1.26 plus VAT EUR 0.25 (BR-037).

### US-014 · See a VAT-inclusive total with a voucher

| Field | Value |
|---|---|
| Epic | EP-03 Ordering, Pickup & Payments |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-ORD-04, FR-PAY-05, FR-PAY-06 |
| Business rules | BR-015, BR-035, BR-036 |
| Dependencies | US-011 |
| Test cases | TC-ORD-004, TC-PAY-003, TC-PAY-004 |

**Story**
As a customer, I want to see the total I will pay, the effect of my voucher and how much VAT the total includes before I pay, so that there are no surprises on my receipt.

**Acceptance criteria**

```gherkin
Scenario: US-014-AC1 Canonical basket with voucher MOHN3
  Given Sophie's cart at Café Mohnblume holds 2 x Topfengolatsche at € 3,40, 1 x Schinken-Käse-Sandwich at € 6,90 and 1 x Hausgemachte Limonade 0,5 l at € 5,00
  When she enters the voucher code "MOHN3"
  Then checkout shows subtotal € 18,70, voucher "MOHN3" − € 3,00 and total € 15,70
  And "darin enthalten" shows USt. 10 %: € 1,05 and USt. 20 %: € 0,70
  And the order button reads "Zahlungspflichtig bestellen · € 15,70"

Scenario: US-014-AC2 The voucher is split across the VAT rates
  Given the basket of AC1 with MOHN3
  Then the quote allocates € 2,20 of the discount to the 10 % group and € 0,80 to the 20 % group
  And the 10 % group is € 11,50 gross, € 10,45 net, € 1,05 VAT
  And the 20 % group is € 4,20 gross, € 3,50 net, € 0,70 VAT

Scenario Outline: US-014-AC3 Minimum order value is measured before the discount
  Given Sophie's basket gross is <gross>
  When she enters "MOHN3"
  Then the result is "<result>"

  Examples:
    | gross   | result                                                      |
    | € 14,99 | Dieser Gutschein gilt ab einem Bestellwert von € 15,00.     |
    | € 15,00 | applied, total € 12,00                                      |

Scenario Outline: US-014-AC4 Voucher rejections
  When Sophie enters "<code>"
  Then she sees "<message>"

  Examples:
    | code     | message                                         |
    | MOHN33   | Diesen Gutscheincode gibt es nicht.             |
    | SOMMER25 | Dieser Gutschein ist derzeit nicht gültig.      |
    | SESAM5   | Dieser Gutschein gilt nicht für Café Mohnblume. |

Scenario: US-014-AC5 Seller identity is shown before paying
  Then above the order button checkout shows "Verkäufer: Mohnblume Kaffeehaus e.U., Kornblumengasse 9, 1070 Wien"
```

**Notes**
- The numbers are the canonical example in [SRS Appendix B.1](../../02-requirements/SRS.md#appendix-b-worked-examples) and business rules table 3.4. Test case TC-PAY-003 reproduces them.
- MOHN3 was valid from 2026-09-01 to 2026-12-31; SOMMER25 expired on 2026-08-31.

### US-015 · Follow my order and collect it

| Field | Value |
|---|---|
| Epic | EP-03 Ordering, Pickup & Payments |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-ORD-05, FR-ORD-06, FR-ORD-07, FR-ORD-09 |
| Business rules | BR-021, BR-022, BR-024, BR-041 |
| Dependencies | US-013, US-026 |
| Test cases | TC-ORD-006, TC-ORD-007, TC-ORD-013 |

**Story**
As a customer, I want to see my order move from placed to ready and pick it up with a code, so that I do not wait at the counter or ask whether it is ready.

**Acceptance criteria**

```gherkin
Scenario: US-015-AC1 Confirmation with pickup code
  When Sophie's order QO-2026-004218 is Placed at 11:53:30
  Then Booking Details shows "Bestellt", the slot "12:15 - 12:30" and the pickup code "K7PM" with a QR code
  And she receives a confirmation email with the order lines and total

Scenario Outline: US-015-AC2 Acceptance timeout of 5 minutes
  Given the order was Placed at 11:53:30
  When Tobias taps "Annehmen" at <time>
  Then the result is "<result>"

  Examples:
    | time     | result                                                                         |
    | 11:58:29 | Accepted; payment captured                                                     |
    | 11:58:30 | refused, the order was auto-rejected with "Keine Antwort vom Geschäft" and voided |

Scenario: US-015-AC3 Live status and push
  Given the order is Accepted
  When Tobias marks it Preparing and then Ready
  Then Booking Details updates within 3 seconds each time without a refresh
  And Sophie receives the push "Café Mohnblume: Deine Bestellung ist fertig"

Scenario: US-015-AC4 A delay is announced
  Given the order is Preparing for slot 12:15
  When Tobias delays it by 15 minutes
  Then Sophie sees "Neue Abholzeit ca. 12:30" and receives a push

Scenario: US-015-AC5 Bookings list
  Given Sophie has one Ready order and two Completed orders
  Then Upcoming shows the Ready order with "Fertig zur Abholung"
  And Past shows the Completed orders, most recent first, each with "Erneut bestellen"
```

### US-016 · Cancel an order and get my money back

| Field | Value |
|---|---|
| Epic | EP-03 Ordering, Pickup & Payments |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-ORD-08, FR-PAY-04 |
| Business rules | BR-023, BR-034 |
| Dependencies | US-015 |
| Test cases | TC-ORD-008, TC-PAY-005 |

**Story**
As a customer whose plans changed, I want to cancel my order in the app while it is still possible, so that I am not charged for food I will not collect.

**Acceptance criteria**

```gherkin
Scenario: US-016-AC1 Cancel while Placed voids the authorization
  Given Sophie's order is Placed with an authorized payment
  When she taps "Bestellung stornieren" and confirms
  Then the order is Canceled, slot capacity is released and the authorization is voided

Scenario Outline: US-016-AC2 Cancel an Accepted order until the cutoff
  Given Sophie's order for slot 12:15 is Accepted and captured, and the cutoff is 30 minutes
  When she tries to cancel at <time>
  Then the result is "<result>"

  Examples:
    | time  | result                                                                        |
    | 11:45 | Canceled; € 15,70 refunded automatically                                      |
    | 11:46 | Cancel not shown; "Bitte ruf Café Mohnblume an: +43 1 5234567"                  |

Scenario: US-016-AC3 Never after preparation started
  Given the order is Preparing at 11:30
  Then the cancel action is not shown, although the cutoff has not passed

Scenario: US-016-AC4 The dialog states how the money comes back
  Given the order is Accepted and captured
  When Sophie taps "Bestellung stornieren"
  Then the dialog reads "Du bekommst € 15,70 auf deine Karte zurück. Die Gutschrift dauert 5 bis 10 Werktage."
```

### US-017 · Rebook a past order

| Field | Value |
|---|---|
| Epic | EP-03 Ordering, Pickup & Payments |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Should |
| Estimate | 3 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-ORD-10 |
| Business rules | BR-025 |
| Dependencies | US-015 |
| Test cases | TC-ORD-009 |

**Story**
As a regular, I want to order the same lunch again with one tap, so that I do not rebuild it item by item.

**Acceptance criteria**

```gherkin
Scenario: US-017-AC1 Rebook copies items at today's prices
  Given Sophie's Completed order had 1 x Linsen-Bowl with "Sesam-Topping" at € 10,30
  And the Linsen-Bowl now costs € 10,20, so the configured line costs € 10,70
  When she taps "Erneut bestellen"
  Then the Café Mohnblume page opens with the cart bar "1 Artikel · € 10,70"
  And she sees "Einige Preise haben sich seit deiner letzten Bestellung geändert."

Scenario: US-017-AC2 Unavailable items are skipped and listed
  Given the past order had 2 items and "Mohnkuchen" is no longer published
  When Sophie rebooks
  Then 1 item is added and she sees "1 Artikel ist nicht mehr verfügbar und wurde nicht hinzugefügt."

Scenario: US-017-AC3 Slot, note, voucher and payment are not copied
  When Sophie rebooks
  Then the checkout has no slot, no note, no voucher and the default payment method

Scenario: US-017-AC4 No rebook on an order still in progress
  Given an order is Preparing
  Then its card shows no "Erneut bestellen"
```

### US-018 · Pre-order a whole cake for a later day

| Field | Value |
|---|---|
| Epic | EP-03 Ordering, Pickup & Payments |
| Persona | PER-02 Daniela Costa (CUS) |
| Priority | Should |
| Estimate | 5 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-ORD-11 |
| Business rules | BR-017 |
| Dependencies | US-012 |
| Test cases | TC-ORD-007, TC-ORD-012 |

**Story**
As a customer planning a birthday, I want to order a whole cake for a later day, so that the bakery can make it in time.

**Acceptance criteria**

```gherkin
Scenario Outline: US-018-AC1 Earliest slot follows the 48-hour lead time
  Given Daniela adds "Schokoladentorte, ganz" (lead time 48 hours) at Konditorei Pomeranze on Monday 2026-09-14 at 10:00
  When she opens slot selection
  Then the slot "<slot>" is <offered>

  Examples:
    | slot                      | offered     |
    | Wed 2026-09-16 09:45      | not offered |
    | Wed 2026-09-16 10:00      | offered     |
    | Mon 2026-09-21 10:00      | offered     |
    | Mon 2026-09-21 10:15      | not offered |

Scenario: US-018-AC2 The largest lead time in the cart governs
  Given the cart holds "Schokoladentorte, ganz" (48 hours) and "Topfentorte (Stück)" (no lead time)
  Then no same-day slot is offered

Scenario: US-018-AC3 A pre-order placed while closed is accepted after opening
  Given Daniela places the pre-order on Monday 2026-09-14 at 20:10 and Pomeranze opens on Tuesday at 06:30
  Then the order's acceptance deadline is Tuesday 2026-09-15 07:00
  And it is not auto-rejected overnight
```

**Notes**
- Added by CR-001 (SRS v1.1) at the request of the pilot bakeries.

### US-019 · Get a receipt in the business's name

| Field | Value |
|---|---|
| Epic | EP-03 Ordering, Pickup & Payments |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-PAY-07 |
| Business rules | BR-038 |
| Dependencies | US-013 |
| Test cases | TC-PAY-006, TC-PAY-010 |

**Story**
As a customer, I want an itemized receipt from the business with the VAT shown, so that I can claim business lunches and check what I paid.

**Acceptance criteria**

```gherkin
Scenario: US-019-AC1 Receipt content for the canonical order
  Given order QO-2026-004218 for € 15,70 was captured
  Then a receipt is emailed and shown under "Beleg" in Booking Details
  And it states seller "Mohnblume Kaffeehaus e.U.", "Kornblumengasse 9, 1070 Wien", UID "ATU71234567"
  And it lists the 3 lines, the voucher − € 3,00, total € 15,70
  And for 10 %: gross € 11,50, net € 10,45, VAT € 1,05; for 20 %: gross € 4,20, net € 3,50, VAT € 0,70
  And it carries "Vermittelt über die Plattform"

Scenario Outline: US-019-AC2 In-app payment is limited to € 400,00
  Given a catering order totals <total>
  When Sophie opens checkout
  Then in-app payment is <state>

  Examples:
    | total    | state                                                                     |
    | € 400,00 | available                                                                 |
    | € 400,01 | unavailable: "In der App kannst du Bestellungen bis € 400,00 bezahlen."    |

Scenario: US-019-AC3 Small-business exemption
  Given Kaffeebar Zimtstern applies the small-business VAT exemption
  When Sophie's order of 2 x Espresso at € 2,60 is captured
  Then the receipt shows total € 5,20 and "Umsatzsteuerbefreit, Kleinunternehmer"
  And it shows no VAT amounts
```

**Notes**
- AC3 covers the Exempt tax category added in SRS v1.2 after a UAT finding at Kaffeebar Zimtstern (BR-015).

### US-020 · Be told what changed before I pay

| Field | Value |
|---|---|
| Epic | EP-03 Ordering, Pickup & Payments |
| Persona | PER-02 Daniela Costa (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S13 / R1.1 |
| Requirements | FR-ORD-12 |
| Business rules | BR-013, BR-019, BR-020 |
| Dependencies | US-013, US-029 |
| Test cases | TC-ORD-010, TC-ORD-011 |

**Story**
As a customer with an allergy, I want checkout to stop me if an item's allergens or price changed after I added it, and to offer me another time if my slot filled while my bank confirmed the payment, so that I never pay for something different from what I chose.

**Acceptance criteria**

```gherkin
Scenario: US-020-AC1 Allergens added since the item was added
  Given Daniela added "Topfentorte (Stück)" at 08:40 when its declaration was version 3 (A C G)
  And the bakery confirmed version 4 (A C G H) at 08:52
  When she taps "Zahlungspflichtig bestellen" at 09:05
  Then payment does not start
  And the line expands showing "Neu: H Schalenfrüchte" in bold
  And she must tick "Neue Allergenangaben gelesen" before the order button is enabled again

Scenario: US-020-AC2 Price changed since the item was added
  Given the Linsen-Bowl was € 9,80 in the cart and is now € 10,20
  When Sophie taps the order button
  Then she sees "Der Preis für „Linsen-Bowl“ hat sich geändert: jetzt € 10,20" and the new total, and must confirm

Scenario: US-020-AC3 The hold is extended once when the bank challenge starts
  Given Sophie's hold on 12:15 was created at 11:40:00 and would expire at 11:50:00
  When the card payment provider reports the 3-D Secure challenge started at 11:43:10
  Then the hold expires at 11:55:00
  And a second challenge start does not extend it again

Scenario: US-020-AC4 The slot filled while the bank confirmed
  Given Sophie's hold on 12:15 expired at 11:55:00 and another order took the last unit at 11:55:30
  When her bank confirms the payment at 11:56:20
  Then no order is created in 12:15
  And she sees "Deine Abholzeit ist inzwischen ausgebucht. Wähle eine neue Zeit, deine Zahlung bleibt reserviert." with 12:30, 12:45 and 13:00 held for 3 minutes
  When she chooses 12:30 at 11:57:40
  Then the order is Placed in 12:30 using the same authorization

Scenario: US-020-AC5 No choice within 3 minutes
  Given alternatives were offered at 11:56:20
  When Sophie has not chosen by 11:59:20
  Then the authorization is voided and she sees "Reservierung aufgehoben, es wurde nichts abgebucht"
```

**Notes**
- AC1 and AC2 implement CR-005 (from [INC-2026-006](../../07-operations/incidents/INC-2026-006-stale-allergen-information.md)); AC3 to AC5 implement CR-006 (from [INC-2026-011](../../07-operations/incidents/INC-2026-011-pickup-slots-over-capacity.md)).
- The re-check runs at the order button and again inside the authorization transaction (business rules tables 3.1c and 3.3).

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [ADR-002 Slot holds](../../03-design/architecture/adr/ADR-002-slot-holds-redis-database-guard.md)
- [ADR-003 Marketplace payments](../../03-design/architecture/adr/ADR-003-marketplace-payments-authorize-capture.md)
- [Wireframe m-02 Checkout and status](../../03-design/wireframes/m-02-checkout-and-status.svg)
- [Change request log](../change-request-log.md)
- [Test cases](../../06-quality/test-cases.md)
