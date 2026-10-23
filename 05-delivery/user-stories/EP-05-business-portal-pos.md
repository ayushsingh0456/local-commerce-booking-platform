# EP-05 Business Portal & POS: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | LC-DEL-US-05 |
| Version | 1.3 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Product Owner, Engineering Lead, QA Lead, UX Designer, Partner Success Lead, Finance Lead |

## Purpose and scope

User stories for EP-05:
- the order board, hand-over, sold-out toggles and pause in the POS view;
- the Business Admin's catalog, allergen declarations, services, staff and working hours;
- refunds and statements.

Version 1.2 added the automatic pause when no board is connected (BR-040). Version 1.3 adds declaration versions and the "live within 1 minute" confirmation (CR-005).

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-05 |
| Name | Business Portal & POS |
| Modules | CAT (Catalog & Business Setup), POS (Order Board & Day Operations (POS view)) |
| Goal | Give the business one portal to maintain its catalog, allergens, services and staff hours, and give counter staff a POS view that runs pickup orders and the appointment day without a phone. |
| Objectives | OBJ-04 Complete and current allergen information; OBJ-06 Free counter staff at lunch |
| Business need | BN-05 |
| Functional requirements | FR-CAT-01 to FR-POS-09 (18) |
| Stories | US-026 to US-032 (7, 40 points) |
| Release | R1 |

## Story list

| Story | Title | Persona | Priority | Points | Sprint | Release |
|---|---|---|---|---|---|---|
| US-026 | Run the order board at the counter | PER-04 Tobias Gruber (BIZ-STF) | Must | 8 | S4 | R1 |
| US-027 | Hand over an order with the pickup code | PER-04 Tobias Gruber (BIZ-STF) | Must | 3 | S5 | R1 |
| US-028 | Mark items sold out and pause orders | PER-04 Tobias Gruber (BIZ-STF) | Must | 3 | S5 | R1 |
| US-029 | Declare allergens for items and choices | PER-03 Elif Aydin (BIZ-ADM) | Must | 8 | S3 | R1 |
| US-030 | Maintain the menu, prices and tax categories | PER-03 Elif Aydin (BIZ-ADM) | Must | 5 | S2 | R1 |
| US-031 | Set up services, staff and working hours | PER-05 Martina Leitner (BIZ-ADM) | Must | 8 | S4 | R1 |
| US-032 | Issue a refund and read my payout statement | PER-03 Elif Aydin (BIZ-ADM) | Must | 5 | S6 | R1 |
| **Total** |  |  |  | **40** |  |  |

Shared test data: Café Mohnblume (L-1001) with Elif Aydin (Business Admin) and Tobias Gruber (Staff, POS tablet at the counter). Haarstudio Linde (L-2001) with Martina Leitner (Business Admin) and bookable staff Lena Auer and Aylin Demir.

## Stories

### US-026 · Run the order board at the counter

| Field | Value |
|---|---|
| Epic | EP-05 Business Portal & POS |
| Persona | PER-04 Tobias Gruber (BIZ-STF) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-POS-01, FR-POS-02, FR-POS-07, FR-POS-09 |
| Business rules | BR-012, BR-022, BR-033, BR-039, BR-040 |
| Dependencies | US-013 |
| Test cases | TC-POS-001, TC-POS-002, TC-POS-006, TC-POS-008 |

**Story**
As counter staff, I want new orders to appear and alert me on one screen with everything I need to prepare them, so that I can accept them quickly without picking up a phone.

**Acceptance criteria**

```gherkin
Scenario: US-026-AC1 New order alert until acted on
  Given the order board for Café Mohnblume is open on the counter tablet
  When order QO-2026-004218 is placed at 11:53:30
  Then within 3 seconds a ticket appears in "Neu" with a sound and a flashing border
  And the ticket shows "Annehmen bis 11:58:30" counting down
  And the sound repeats every 10 seconds until Tobias accepts or rejects

Scenario: US-026-AC2 The ticket shows what the kitchen needs
  Then the ticket shows "Sophie", "Abholung 12:15", "2 x Topfengolatsche (A C G)", "1 x Schinken-Käse-Sandwich (A G M)", "1 x Hausgemachte Limonade · 0,5 l · Holunder" and "In-App-Zahlung"

Scenario: US-026-AC3 Configured lines show the allergens of the configuration
  Given an order line "Linsen-Bowl · Cashew-Topping"
  Then the ticket shows the allergens "L H" for that line

Scenario: US-026-AC4 Accept captures, reject needs a reason
  When Tobias taps "Annehmen"
  Then the order moves to "In Zubereitung" column as Accepted and the payment is captured
  When on another order he taps "Ablehnen" without choosing a reason
  Then the action is refused with "Bitte wählen Sie einen Grund."

Scenario: US-026-AC5 Staff cannot open Business Admin functions
  Given Tobias is signed in with the Staff role
  Then the portal shows only "Bestellungen", "Ausverkauft", "Pause" and "Termine heute"
  And a direct call to PUT /business/items/{itemId} with his token returns 403 FORBIDDEN_FOR_ROLE
```

**Notes**
- Tickets show the customer's first name only; the phone number is one tap away for contacting the customer about this order (compliance mapping D9).
- The board layout is wireframe [w-01](../../03-design/wireframes/w-01-pos-order-board.svg).

### US-027 · Hand over an order with the pickup code

| Field | Value |
|---|---|
| Epic | EP-05 Business Portal & POS |
| Persona | PER-04 Tobias Gruber (BIZ-STF) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-POS-03, FR-POS-08 |
| Business rules | BR-021, BR-024 |
| Dependencies | US-026 |
| Test cases | TC-POS-003, TC-POS-004, TC-POS-009 |

**Story**
As counter staff, I want to match the customer's pickup code before I hand over, so that the right person gets the right bag.

**Acceptance criteria**

```gherkin
Scenario: US-027-AC1 Hand over with the matching code
  Given QO-2026-004218 is Ready with pickup code "K7PM"
  When Sophie shows "K7PM" and Tobias types it
  Then the order is Completed

Scenario: US-027-AC2 Wrong code
  When Tobias types "K7PN"
  Then he sees "Code stimmt nicht. Bitte noch einmal prüfen." and the order stays Ready
  And after 5 wrong codes within 10 minutes hand-over for that order is locked for 5 minutes

Scenario: US-027-AC3 Pay-at-pickup needs the counter payment confirmation
  Given a Ready order with "Bezahlung bei Abholung" for € 9,80
  When Tobias matches the code without ticking "Bezahlt an der Kassa"
  Then hand-over is refused with "Bitte bestätigen Sie die Zahlung an der Kassa."
  When he takes the payment on the café's own register and ticks the box
  Then the order is Completed and its payment state is "Paid at counter"

Scenario Outline: US-027-AC4 Not collected from 60 minutes after the slot ends
  Given an order for slot 12:15 - 12:30 is Ready
  When Tobias taps "Nicht abgeholt" at <time>
  Then the result is "<result>"

  Examples:
    | time  | result                   |
    | 13:29 | "Erst ab 13:30 möglich." |
    | 13:30 | Not collected            |
```

**Notes**
- The platform records only that the counter payment happened; amount and method stay in the business's certified register (CR-003 rejected, compliance mapping T6).

### US-028 · Mark items sold out and pause orders

| Field | Value |
|---|---|
| Epic | EP-05 Business Portal & POS |
| Persona | PER-04 Tobias Gruber (BIZ-STF) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-POS-04, FR-POS-05 |
| Business rules | BR-014, BR-040 |
| Dependencies | US-026 |
| Test cases | TC-POS-005, TC-POS-007 |

**Story**
As counter staff, I want to switch off items that ran out and pause new orders when we are overwhelmed, so that customers cannot order what we cannot deliver.

**Acceptance criteria**

```gherkin
Scenario Outline: US-028-AC1 Sold out until end of day or until reset
  Given Tobias marks "Mohnkuchen" sold out "<mode>" at 14:10 on Thursday
  When the café opens on Friday at 07:30
  Then "Mohnkuchen" is <state>

  Examples:
    | mode            | state                 |
    | bis Tagesende   | available again       |
    | bis auf Widerruf | still sold out       |

Scenario: US-028-AC2 Customers see sold out within 60 seconds
  When Tobias marks "Topfengolatsche" sold out at 11:20:00
  Then the app shows it as "Ausverkauft" by 11:21:00
  And carts that contain it cannot be checked out until the line is removed

Scenario Outline: US-028-AC3 Pause new orders
  When Tobias pauses orders for <duration> at 12:05
  Then no pickup slots are offered until <until>
  And orders already placed are unaffected

  Examples:
    | duration                 | until              |
    | 15 minutes               | 12:20              |
    | 60 minutes               | 13:05              |
    | until resumed            | Tobias resumes     |

Scenario: US-028-AC4 Automatic pause when no board is connected
  Given Café Mohnblume accepts orders and the counter tablet loses power at 10:00:00
  When no order board session is connected by 10:02:00
  Then ordering pauses automatically with the reason "no_board_connected"
  And Elif receives an email "Bestellungen pausiert: keine Bestelltafel verbunden"
```

### US-029 · Declare allergens for items and choices

| Field | Value |
|---|---|
| Epic | EP-05 Business Portal & POS |
| Persona | PER-03 Elif Aydin (BIZ-ADM) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-CAT-04, FR-CAT-05 |
| Business rules | BR-010, BR-011, BR-012, BR-013 |
| Dependencies | US-030 |
| Test cases | TC-CAT-002, TC-CAT-003, TC-CAT-004 |

**Story**
As a café owner, I want the portal to make me declare allergens completely for every item and choice and confirm each change, so that my customers can trust what the app shows.

**Acceptance criteria**

```gherkin
Scenario: US-029-AC1 No publication without a complete declaration
  Given Elif creates "Nussschnecke" without selecting allergens and without "Enthält keines der 14 Allergene"
  When she taps "Veröffentlichen"
  Then publishing is refused with "Bitte wählen Sie die enthaltenen Allergene aus oder bestätigen Sie „Enthält keines der 14 Allergene“."

Scenario: US-029-AC2 Every choice needs its own declaration
  Given "Linsen-Bowl" declares L and its option group "Topping" has "Sesam-Topping" declared N and "Cashew-Topping" undeclared
  When Elif taps "Veröffentlichen"
  Then publishing is refused with "Die Auswahl „Cashew-Topping“ hat noch keine Allergenangaben."

Scenario: US-029-AC3 "Contains none" is a declaration that must be confirmed
  When Elif ticks "Enthält keines der 14 Allergene" for "Hausgemachte Limonade"
  Then the allergen list is cleared and disabled
  And publishing requires the confirmation "Ich bestätige, dass die Allergenangaben der aktuellen Rezeptur entsprechen."

Scenario: US-029-AC4 A change creates a new version, live within 1 minute
  Given "Nussschnecke" is published with declaration version 2 (A C G H)
  When Elif adds "F Soja" and confirms at 21:15:00
  Then version 3 (A C F G H) is created with Elif as confirmer and an audit event "allergen.confirmed"
  And the editor shows "Live in der App in spätestens 1 Minute · Version 3"
  And until she confirms, customers keep seeing version 2

Scenario: US-029-AC5 Open pre-orders are told about the change
  Given a customer pre-ordered 4 x Nussschnecke for Saturday
  When Elif confirms version 3
  Then that customer is emailed within 5 minutes with the new allergens and a free cancellation link
```

**Notes**
- AC4 and AC5 were revised by CR-005 after [INC-2026-006](../../07-operations/incidents/INC-2026-006-stale-allergen-information.md).
- The editor is wireframe [w-02](../../03-design/wireframes/w-02-item-editor-allergens.svg).

### US-030 · Maintain the menu, prices and tax categories

| Field | Value |
|---|---|
| Epic | EP-05 Business Portal & POS |
| Persona | PER-03 Elif Aydin (BIZ-ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-CAT-03, FR-CAT-09 |
| Business rules | BR-014, BR-015, BR-036 |
| Dependencies | US-034 |
| Test cases | TC-CAT-001, TC-CAT-009 |

**Story**
As a café owner, I want to maintain my menu with prices including VAT and the right tax category, and create my own vouchers, so that customers see correct prices and receipts show correct VAT.

**Acceptance criteria**

```gherkin
Scenario Outline: US-030-AC1 Price range
  When Elif enters the price <price> for an item
  Then the result is "<result>"

  Examples:
    | price    | result                                                              |
    | 0,09     | Bitte geben Sie einen Preis zwischen € 0,10 und € 999,99 ein.        |
    | 0,10     | accepted                                                            |
    | 999,99   | accepted                                                            |
    | 1000,00  | Bitte geben Sie einen Preis zwischen € 0,10 und € 999,99 ein.        |

Scenario: US-030-AC2 Tax category is required
  When Elif saves "Hausgemachte Limonade" without a tax category
  Then she sees "Bitte wählen Sie die Steuerkategorie."
  When she chooses "Normalsteuersatz 20 %"
  Then the item saves

Scenario: US-030-AC3 Hidden items stay in the catalog
  When Elif sets "Mitarbeiter-Frühstück" to "In der App ausblenden"
  Then it disappears from the app and search within 60 seconds and stays in the portal

Scenario: US-030-AC4 Create a business-funded voucher
  When Elif creates "MOHN3" as € 3,00 off from € 15,00, valid 01.09.2026 to 31.12.2026, once per customer, 200 redemptions in total
  Then the voucher is active and funded by Café Mohnblume
  And its discounts reduce the commission base (BR-037)
```

### US-031 · Set up services, staff and working hours

| Field | Value |
|---|---|
| Epic | EP-05 Business Portal & POS |
| Persona | PER-05 Martina Leitner (BIZ-ADM) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-CAT-06, FR-CAT-07, FR-CAT-08 |
| Business rules | BR-026, BR-031, BR-039 |
| Dependencies | US-034 |
| Test cases | TC-CAT-006, TC-CAT-007, TC-CAT-008 |

**Story**
As a salon owner, I want to set up services with their buffers, my team and their working hours, without ever leaving a booked customer without a stylist, so that online availability matches reality.

**Acceptance criteria**

```gherkin
Scenario Outline: US-031-AC1 Service duration and buffer ranges
  When Martina saves a service with duration <duration> and buffer <buffer>
  Then the result is "<result>"

  Examples:
    | duration | buffer | result                                   |
    | 10       | 0      | refused: duration from 15 to 480 minutes |
    | 45       | 15     | accepted                                 |
    | 47       | 15     | refused: duration in 5-minute steps      |
    | 45       | 61     | refused: buffer from 0 to 60 minutes     |

Scenario: US-031-AC2 Invite staff with a role
  When Martina invites "lena.auer@haarstudio-linde.example" as Staff and marks her bookable
  Then Lena receives an invitation and appears as bookable for her qualified services once she accepts

Scenario: US-031-AC3 Deactivation ends sessions at once
  When Martina deactivates a former employee who is signed in on the POS tablet
  Then that session ends at its next request

Scenario: US-031-AC4 A working-hours change cannot orphan appointments
  Given Lena has Sophie's appointment on 2026-10-13 at 09:30
  When Martina changes Lena's Tuesday hours to start at 11:00
  Then saving is refused with "1 Termine sind betroffen. Bitte umbuchen oder absagen." listing Sophie's appointment
  When she cancels it with the reason "Dienstplanänderung", or reassigns it to a qualified staff member who is free from 09:30 to 10:30
  Then the change saves and Sophie is notified of the cancellation or the new stylist
```

**Notes**
- AC4 prevents the scenario raised in the UAT script UAT-07, where a salon owner shortened a shift in a paper diary and a customer arrived to no stylist.
- The message uses the plural rules of the translation file ("1 Termin ist betroffen").

### US-032 · Issue a refund and read my payout statement

| Field | Value |
|---|---|
| Epic | EP-05 Business Portal & POS |
| Persona | PER-03 Elif Aydin (BIZ-ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S6 / R1 |
| Requirements | FR-PAY-04, FR-PAY-08 |
| Business rules | BR-023, BR-034, BR-037 |
| Dependencies | US-013 |
| Test cases | TC-PAY-005, TC-PAY-008 |

**Story**
As a café owner, I want to refund a customer in full or in part and see what was paid out to me and why, so that I can fix mistakes and reconcile my bank account.

**Acceptance criteria**

```gherkin
Scenario: US-032-AC1 Partial refund split across VAT rates
  Given order QO-2026-004218 was captured for € 15,70 (10 % group € 11,50, 20 % group € 4,20)
  When Elif refunds € 5,00 with the reason "Falsche Limonadensorte, Golatsche fehlte"
  Then the refund is allocated € 3,66 to 10 % and € 1,34 to 20 %, with VAT reversed € 0,33 and € 0,22
  And € 0,40 commission plus € 0,08 VAT is reversed

Scenario Outline: US-032-AC2 Refunds never exceed the captured amount
  Given € 5,00 has already been refunded on the € 15,70 order
  When Elif refunds <amount>
  Then the result is "<result>"

  Examples:
    | amount   | result                              |
    | € 10,70  | accepted; payment fully refunded    |
    | € 10,71  | "Höchstens € 10,70 erstattbar."      |

Scenario: US-032-AC3 Staff cannot refund
  Given Tobias is signed in as Staff
  Then the order detail shows no "Erstatten" action
  And POST /business/orders/{orderId}/refunds with his token returns 403

Scenario: US-032-AC4 Payout statement
  Given in the week of 2026-09-14 the only order was the canonical € 15,70 order
  When Elif opens the payout statement for that week
  Then it shows captured € 15,70, commission € 1,26, VAT on commission € 0,25 and payout € 14,19
```

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [Wireframe w-01 POS order board](../../03-design/wireframes/w-01-pos-order-board.svg)
- [Wireframe w-02 Item editor with allergens](../../03-design/wireframes/w-02-item-editor-allergens.svg)
- [Realtime events](../../04-api/realtime-events.md)
- [Test cases](../../06-quality/test-cases.md)
