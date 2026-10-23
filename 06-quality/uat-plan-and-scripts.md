# UAT Plan and Scripts: Platform release 1

## Document control

| Field | Value |
|---|---|
| Document ID | QT-QA-03 |
| Version | 1.2 |
| Status | Signed off |
| Owner | Business Analyst (UAT facilitator) |
| Last updated | 2026-06-26 |
| Reviewers | Product Owner; QA Lead; Partner Success Lead |

**Purpose and scope.** How the pilot businesses and a customer panel accepted the platform R1 before the pilot go-live, the 10 scripts they ran and the sign-off. UAT checks that the product supports the businesses' real work; it is not a second system test, and it assumes the [system test](test-strategy-and-plan.md) exit criteria are met.

## 1. Approach

| Item | Plan |
|---|---|
| Dates | 2026-06-15 to 2026-06-26 (two weeks), on staging build 1.0.0-rc.4 |
| Participants | 6 pilot businesses: Café Mohnblume (Elif Aydin, Tobias Gruber), Konditorei Pomeranze, Nudelbar Sesam, Haarstudio Linde (Martina Leitner and two stylists), Kaffeebar Zimtstern, Massageatelier Ruhepol. A customer panel of 12 residents of Neubau and Mariahilf (4 with food allergies, 3 using English). Partner Operations (Paul Haider). |
| Setting | Scripts 02, 05 and 06 ran in Café Mohnblume during a real lunch service, on the café's own tablet, with test orders from the panel. Others ran at the businesses or in the platform's office. |
| Data | UAT data set based on [test data](test-data/README.md), with voucher validity moved into the UAT window (MOHN3 valid 2026-06-15 to 2026-06-26); panel members used their own phones with sandbox cards |
| Facilitation | The BA observes and records; participants act. No coaching beyond the script's starting point. |
| Defects | Logged with the script ID; severity agreed with the participant; Critical or High blocks sign-off |

**Entry criteria.** System test exit met; no open Critical or High defect; participants trained for 30 minutes; UAT data loaded; virtual clock available for scripts that depend on time.

**Exit criteria.** Every script passed, or passed with a Medium or Low defect accepted by the participant and the Product Owner; sign-off by each business and the panel representative.

## 2. Scripts

Each script lists the participant, the steps, what to observe and the result in the UAT cycle.

### UAT-01 Sign up, browse and check allergens (customer panel)

| Step | Action | Expected |
|---|---|---|
| 1 | Browse Home and a menu without an account | Prices and allergens visible; Add asks for sign-in |
| 2 | Sign up with your own details | Only name, email, mobile, password and the age box are asked |
| 3 | Open three items you would order and read the allergens | Allergens above the options, in letters and words |
| 4 | Panel members with allergies: set the filter for your allergen | Items marked; nothing says "safe" |

Result: Passed. Two of four allergy participants asked for traces information; logged as TBD-02 (R2 consideration).

### UAT-02 Lunch order with voucher, bank confirmation and pickup (customer panel, Café Mohnblume)

| Step | Action | Expected |
|---|---|---|
| 1 | Order 2 Topfengolatsche, 1 Schinken-Käse-Sandwich, 1 Limonade 0,5 l | Cart € 18,70 |
| 2 | Enter MOHN3 | Total € 15,70; VAT 10 % € 1,05 and 20 % € 0,70 |
| 3 | Pay with the sandbox card ending 3155 and approve the challenge | Order Placed with a pickup code |
| 4 | Watch the status while Tobias accepts and prepares | Status updates without refreshing |
| 5 | Collect with the code | Completed; receipt by email with the same VAT figures |

Result: Passed. The canonical worked example reproduced exactly on 3 of 3 runs.

### UAT-03 Cancel, refund and rebook (customer panel)

| Step | Action | Expected |
|---|---|---|
| 1 | Cancel a Placed order | Nothing charged |
| 2 | Cancel an Accepted order before the cutoff | Refund message with 5 to 10 working days |
| 3 | Try to cancel after the cutoff | Phone number shown instead |
| 4 | Rebook a completed order | Items in the cart at today's prices |

Result: Passed.

### UAT-04 Book, get reminded, reschedule (customer panel, Haarstudio Linde)

| Step | Action | Expected |
|---|---|---|
| 1 | Book Damenhaarschnitt with Lena on Tuesday | Only times that fit are offered (in the UAT data: 09:00, 09:15, 09:30, 11:15, 11:30, 11:45, 12:00) |
| 2 | Book 09:30 with a note | Confirmation with calendar file |
| 3 | Advance the clock to 24 hours before | Reminder by email and push |
| 4 | Reschedule to 11:30 | Old appointment replaced in one step |
| 5 | Salon marks the original slot no-show (stylist's idea) | Must not be possible before the start plus 15 minutes |

Result: Passed after fix. Step 5 found DEF-066 (no-show allowed before the start); fixed in rc.4 and re-run on 2026-06-23.

### UAT-05 Lunch rush on the order board (Café Mohnblume staff)

| Step | Action | Expected |
|---|---|---|
| 1 | Run the board on the counter tablet during lunch, with 25 panel orders between 11:45 and 12:45 | Every order noticed; median time to accept 60 seconds or less |
| 2 | Mark Topfengolatsche sold out | Disappears from ordering within a minute |
| 3 | Pause orders for 15 minutes when the queue builds | No new slots; existing orders unaffected |
| 4 | Hand over with codes | Wrong codes rejected |

Result: Passed after fix. The median time to accept was 41 seconds. DEF-089 (sound not repeating on iPad after the screen locked) was fixed and re-tested on 2026-06-24.

### UAT-06 What happens when the board goes offline (Café Mohnblume staff)

| Step | Action | Expected |
|---|---|---|
| 1 | Switch off the tablet's Wi-Fi for 1 minute | Board shows "Live-Verbindung unterbrochen" and catches up afterwards |
| 2 | Close every board session for 3 minutes during ordering hours | Orders must not keep arriving unseen |

Result: Passed with a clarification. Step 2 showed that orders kept arriving with nobody watching. This produced BR-040's automatic pause after 2 minutes (SRS v1.2), re-tested on 2026-06-25.

### UAT-07 Change working hours without stranding a customer (Haarstudio Linde)

| Step | Action | Expected |
|---|---|---|
| 1 | Shorten Lena's Tuesday to start at 11:00 although she has a 09:30 appointment | Save refused with the affected appointment listed |
| 2 | Reassign or cancel it with a reason | Saved; the customer is told |

Result: Passed. Martina Leitner: "That is exactly what happens with the paper diary."

### UAT-08 Menu, allergens and refunds (Café Mohnblume and Kaffeebar Zimtstern owners)

| Step | Action | Expected |
|---|---|---|
| 1 | Create Linsen-Bowl with two toppings and declare allergens | Publishing blocked until every choice is declared |
| 2 | Declare "contains none" for Limonade | Confirmation required |
| 3 | Kaffeebar Zimtstern: set up as a small business without UID | A tax category without VAT is available |
| 4 | Refund € 5,00 on a captured order | Refund and VAT split shown |

Result: Passed after fix. Step 3 found DEF-071 (no Exempt category), fixed in rc.4 (BR-015, SRS v1.2).

### UAT-09 From application to Live (Konditorei Pomeranze and Partner Operations)

| Step | Action | Expected |
|---|---|---|
| 1 | Apply with the bakery's real register data (masked in the report) | UID check result shown |
| 2 | Paul verifies and approves | Decision recorded with reason |
| 3 | Complete payment onboarding in the provider's hosted flow | Checklist shows "Zahlungen: eingerichtet" |
| 4 | Publish one item and request go-live | Live only when the gate passes |

Result: Passed. Elapsed time 3 business days in UAT conditions.

### UAT-10 Suspend, change a VAT rate, read statements (Partner Operations, Finance Lead, Café Mohnblume)

| Step | Action | Expected |
|---|---|---|
| 1 | Suspend a test business with a statement of reasons | Delisted; open orders continue; statement emailed |
| 2 | Add a future-dated test VAT rate in staging | Applies by order date |
| 3 | Café owner opens the weekly statement | Captured, commission, VAT on commission and payout reconcile |

Result: Passed.

## 3. Results summary

| Script | Result | Defects |
|---|---|---|
| UAT-01 | Passed | None (TBD-02 logged) |
| UAT-02 | Passed | None |
| UAT-03 | Passed | None |
| UAT-04 | Passed after fix | DEF-066 Medium, fixed |
| UAT-05 | Passed after fix | DEF-089 High, fixed |
| UAT-06 | Passed with clarification | BR-040 auto-pause added |
| UAT-07 | Passed | None |
| UAT-08 | Passed after fix | DEF-071 Medium, fixed |
| UAT-09 | Passed | None |
| UAT-10 | Passed | None |

## 4. Sign-off

| Participant | Role | Decision | Date |
|---|---|---|---|
| Café Mohnblume (Elif Aydin) | Pilot food business | Accepted | 2026-06-26 |
| Konditorei Pomeranze | Pilot food business | Accepted | 2026-06-26 |
| Nudelbar Sesam | Pilot food business | Accepted | 2026-06-26 |
| Kaffeebar Zimtstern | Pilot food business | Accepted | 2026-06-26 |
| Haarstudio Linde (Martina Leitner) | Pilot salon | Accepted | 2026-06-26 |
| Massageatelier Ruhepol | Pilot spa | Accepted | 2026-06-26 |
| Customer panel representative | Customers | Accepted | 2026-06-26 |
| Product Owner | the operator | Accepted for pilot go-live | 2026-06-26 |

## Related documents

- [Test strategy and plan](test-strategy-and-plan.md)
- [Test cases](test-cases.md)
- [Test data](test-data/README.md)
- [Release and sprint plan](../05-delivery/release-and-sprint-plan.md)
- [Software Requirements Specification](../02-requirements/SRS.md)
