# EP-04 Appointment Booking: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DEL-US-04 |
| Version | 1.2 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-06-19 |
| Reviewers | Product Owner, Engineering Lead, QA Lead, UX Designer, Partner Success Lead |

## Purpose and scope

User stories for EP-04: choosing a service and staff member, computing start times with buffers, holding and confirming, reminders, self-service changes, and the business side of the appointment day. Version 1.2 aligns US-025 with the UAT clarification that a no-show can be set only from 15 minutes after the start (BR-030). Spa classes and events are out of scope (CR-002).

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-04 |
| Name | Appointment Booking |
| Modules | APT (Appointment Booking) |
| Goal | Let a customer book a salon or spa appointment around the clock, offering only start times that really fit the staff member's day, with reminders and self-service changes that cut no-shows. |
| Objectives | OBJ-02 Cut appointment no-shows; OBJ-03 Take bookings when the phone cannot |
| Business need | BN-04 |
| Functional requirements | FR-APT-01 to FR-APT-10 (10) |
| Stories | US-021 to US-025 (5, 26 points) |
| Release | R1 |

## Story list

| Story | Title | Persona | Priority | Points | Sprint | Release |
|---|---|---|---|---|---|---|
| US-021 | Choose a service and a staff member | PER-01 Sophie Wallner (CUS) | Must | 5 | S4 | R1 |
| US-022 | See only start times that really fit | PER-01 Sophie Wallner (CUS) | Must | 8 | S5 | R1 |
| US-023 | Hold and confirm my appointment | PER-01 Sophie Wallner (CUS) | Must | 5 | S5 | R1 |
| US-024 | Get a reminder and reschedule or cancel | PER-01 Sophie Wallner (CUS) | Must | 5 | S5 | R1 |
| US-025 | Check in, complete or mark a no-show | PER-05 Martina Leitner (BIZ-ADM) | Must | 3 | S6 | R1 |
| **Total** |  |  |  | **26** |  |  |

Shared test data ([salon-schedule.csv](../../06-quality/test-data/salon-schedule.csv)): Haarstudio Linde (L-2001), minimum notice 2 hours, booking horizon 8 weeks, free cancellation until 24 hours before the start.

| Service | Duration | Buffer after | Price | Staff |
|---|---|---|---|---|
| Damenhaarschnitt (Waschen, Schneiden, Föhnen) | 45 min | 15 min | EUR 52.00 | Lena Auer, Aylin Demir |
| Herrenhaarschnitt | 30 min | 15 min | EUR 32.00 | Lena Auer, Aylin Demir |
| Ansatzfarbe | 60 min | 15 min | from EUR 65.00 | Aylin Demir |

On Tuesday 2026-10-13 Lena works 09:00 to 13:00 and has one appointment at 10:30 (Herrenhaarschnitt, occupying 10:30 to 11:15). Aylin works 13:00 to 18:00 with no appointments.

## Stories

### US-021 · Choose a service and a staff member

| Field | Value |
|---|---|
| Epic | EP-04 Appointment Booking |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-APT-01, FR-APT-02 |
| Business rules | BR-015, BR-026, BR-027 |
| Dependencies | US-031 |
| Test cases | TC-APT-001 |

**Story**
As a customer, I want to pick the service I need and either my usual stylist or whoever is free, so that I can book the way I prefer.

**Acceptance criteria**

```gherkin
Scenario: US-021-AC1 Service list
  When Sophie opens Haarstudio Linde
  Then the service list shows "Damenhaarschnitt (Waschen, Schneiden, Föhnen) · 45 Min. · € 52,00"
  And "Ansatzfarbe · 60 Min. · ab € 65,00"

Scenario: US-021-AC2 Only qualified staff can be chosen
  When Sophie chooses "Ansatzfarbe"
  Then the staff choices are "Aylin Demir" and "Beliebig"
  And Lena Auer is not offered

Scenario: US-021-AC3 Beliebig shows the union of start times
  When Sophie chooses "Damenhaarschnitt" with "Beliebig" for 2026-10-13, viewed on 2026-10-12 at 18:00
  Then the start times are Lena's 09:00, 09:15, 09:30, 11:15, 11:30, 11:45, 12:00 and Aylin's 13:00 to 17:00 every 15 minutes
```

### US-022 · See only start times that really fit

| Field | Value |
|---|---|
| Epic | EP-04 Appointment Booking |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-APT-02 |
| Business rules | BR-026 |
| Dependencies | US-021 |
| Test cases | TC-APT-002, TC-APT-003 |

**Story**
As a customer, I want to see only start times at which my whole service, including the stylist's cleanup time, fits, so that I never book a time the salon has to move.

**Acceptance criteria**

```gherkin
Scenario: US-022-AC1 Canonical start times with buffer
  Given Lena works 09:00 to 13:00 on 2026-10-13 and has an appointment occupying 10:30 to 11:15
  When Sophie views "Damenhaarschnitt" (45 minutes plus 15 minutes buffer) with Lena on 2026-10-12 at 18:00
  Then exactly these 7 start times are offered: 09:00, 09:15, 09:30, 11:15, 11:30, 11:45, 12:00

Scenario Outline: US-022-AC2 Boundaries of the occupied interval and the shift
  Given the situation of AC1
  Then the start time <start> is <result> because <reason>

  Examples:
    | start | result      | reason                                                      |
    | 09:30 | offered     | it ends at 10:30, where the existing appointment begins     |
    | 09:45 | not offered | it overlaps 10:30 to 11:15                                  |
    | 11:00 | not offered | it overlaps the buffer that ends at 11:15                   |
    | 11:15 | offered     | it starts exactly when the buffer ends                      |
    | 12:00 | offered     | its buffer ends at 13:00, the end of the shift              |
    | 12:15 | not offered | the service ends at 13:00 but its buffer runs to 13:15      |

Scenario: US-022-AC3 Booking a time removes overlapping start times
  Given the situation of AC1
  When Sophie books 09:30
  Then Lena's start times for the service are 11:15, 11:30, 11:45, 12:00
  And 09:00 and 09:15 are no longer offered

Scenario Outline: US-022-AC4 Minimum notice of 2 hours
  Given it is 2026-10-13 at <now>
  When Sophie views Lena's start times for the same day
  Then the first start time offered is <first>

  Examples:
    | now   | first |
    | 07:15 | 09:15 |
    | 07:16 | 09:30 |

Scenario Outline: US-022-AC5 Booking horizon of 8 weeks
  Given it is 2026-10-12
  When Sophie opens the date <date>
  Then the date is <state>

  Examples:
    | date       | state        |
    | 2026-12-07 | bookable     |
    | 2026-12-08 | not bookable |
```

**Notes**
- The numbers are the canonical example in [SRS Appendix B.2](../../02-requirements/SRS.md#appendix-b-worked-examples) and business rules table 3.5. Test case TC-APT-002 reproduces them.
- Seven start times do not mean seven appointments: the free time allows only 2 more Damenhaarschnitt appointments. Gap-minimizing suggestions are an R2 idea.

### US-023 · Hold and confirm my appointment

| Field | Value |
|---|---|
| Epic | EP-04 Appointment Booking |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-APT-03, FR-APT-04, FR-APT-08, FR-APT-09 |
| Business rules | BR-027, BR-028, BR-032 |
| Dependencies | US-022 |
| Test cases | TC-APT-004, TC-APT-005, TC-APT-006 |

**Story**
As a customer, I want my chosen time kept for me while I confirm, and a confirmation I can put in my calendar, so that I know the appointment is mine.

**Acceptance criteria**

```gherkin
Scenario: US-023-AC1 Hold and confirm
  Given Sophie selects Lena at 09:30 on 2026-10-13 at 18:02 on 2026-10-12
  Then the screen shows "Termin reserviert · 10:00"
  And above the button "€ 52,00 · Bezahlung vor Ort"
  When she adds the note "Bitte nur Spitzen schneiden" and taps "Zahlungspflichtig buchen"
  Then the appointment is Confirmed for 09:30 to 10:15
  And she receives a confirmation email with a calendar file

Scenario Outline: US-023-AC2 Hold expiry
  Given Sophie selected the time at 18:02:00
  When she taps "Zahlungspflichtig buchen" at <time>
  Then the result is "<result>"

  Examples:
    | time     | result                                                                      |
    | 18:11:59 | Confirmed                                                                   |
    | 18:12:00 | "Deine Reservierung ist abgelaufen. Bitte wähle die Uhrzeit erneut."        |

Scenario: US-023-AC3 Two customers, one time
  Given two requests to confirm Lena at 09:30 reach the server at the same moment
  Then exactly one appointment is Confirmed
  And the other customer sees "Dieser Termin ist leider nicht mehr frei. Nächste freie Zeiten: 11:15, 11:30, 11:45."

Scenario: US-023-AC4 Beliebig assigns the staff member with fewer booked minutes
  Given on 2026-10-14 Lena has 120 booked minutes and Aylin 60, and both can do 14:00
  When Sophie books "Herrenhaarschnitt" at 14:00 with "Beliebig"
  Then Aylin is assigned

Scenario Outline: US-023-AC5 Note length
  When Sophie enters a note of <length> characters
  Then the result is "<result>"

  Examples:
    | length | result                                         |
    | 300    | accepted                                       |
    | 301    | Bitte kürze deine Anmerkung auf 300 Zeichen.   |
```

### US-024 · Get a reminder and reschedule or cancel

| Field | Value |
|---|---|
| Epic | EP-04 Appointment Booking |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-APT-05, FR-APT-06 |
| Business rules | BR-029, BR-041 |
| Dependencies | US-023 |
| Test cases | TC-APT-007, TC-APT-008 |

**Story**
As a customer, I want a reminder the day before and the option to move or cancel myself, so that I do not forget and do not have to call during opening hours.

**Acceptance criteria**

```gherkin
Scenario: US-024-AC1 Reminder 24 hours before
  Given Sophie's appointment is on 2026-10-13 at 09:30
  Then at 2026-10-12 09:30 she receives an email and a push "Morgen 09:30: Damenhaarschnitt bei Haarstudio Linde"
  And both contain links to reschedule or cancel

Scenario: US-024-AC2 No reminder for a short-notice booking
  Given an appointment for 2026-10-13 at 15:00 was booked on 2026-10-13 at 08:00
  Then no reminder is sent, only the confirmation

Scenario Outline: US-024-AC3 Free cancellation until 24 hours before
  Given the appointment is on 2026-10-13 at 09:30
  When Sophie tries to cancel on 2026-10-12 at <time>
  Then the result is "<result>"

  Examples:
    | time  | result                                                                                       |
    | 09:30 | Canceled free of charge                                                                     |
    | 09:31 | "Online-Änderungen sind bis 24 Stunden vor dem Termin möglich. Bitte ruf Haarstudio Linde an: +43 1 5267788." |

Scenario: US-024-AC4 Reschedule is atomic
  Given Sophie reschedules her 09:30 appointment to 11:30 on the same day
  When the new time is confirmed
  Then the 09:30 appointment is Canceled and the 11:30 appointment is Confirmed in the same transaction
  When instead 11:30 is taken before she confirms
  Then her 09:30 appointment stays Confirmed

Scenario: US-024-AC5 Reminders do not depend on push
  Given Sophie switched push notifications off
  Then the reminder still arrives by email
```

### US-025 · Check in, complete or mark a no-show

| Field | Value |
|---|---|
| Epic | EP-04 Appointment Booking |
| Persona | PER-05 Martina Leitner (BIZ-ADM) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S6 / R1 |
| Requirements | FR-APT-07, FR-APT-10, FR-POS-06 |
| Business rules | BR-030, BR-031 |
| Dependencies | US-023 |
| Test cases | TC-APT-009, TC-APT-010, TC-POS-010 |

**Story**
As a salon owner, I want to run the appointment day from the POS view and record who came, so that the diary stays true and no-shows are counted fairly.

**Acceptance criteria**

```gherkin
Scenario: US-025-AC1 Day view
  When Martina opens the appointment day view for 2026-10-13
  Then Lena's column shows 09:30 Sophie W. "Damenhaarschnitt" with the note "Bitte nur Spitzen schneiden" and 10:30 "Herrenhaarschnitt"

Scenario: US-025-AC2 Check in and complete
  Given Sophie arrives at 09:27
  When Martina taps "Eingecheckt" and later "Abgeschlossen"
  Then the appointment moves Confirmed to Checked in to Completed and Sophie's app shows "Abgeschlossen"

Scenario Outline: US-025-AC3 No-show only from 15 minutes after the start
  Given the 09:30 appointment is Confirmed and the customer has not arrived
  When Martina taps "Nicht erschienen" at <time>
  Then the result is "<result>"

  Examples:
    | time  | result                    |
    | 09:44 | "Erst ab 09:45 möglich."  |
    | 09:45 | No-show recorded          |

Scenario: US-025-AC4 Business cancellation needs a reason and informs the customer
  Given Lena is ill on 2026-10-13
  When Martina cancels Sophie's 09:30 appointment with the reason "Mitarbeiterin erkrankt"
  Then Sophie receives a push and an email with the reason and a link to book again
  And canceling without a reason is refused with "Bitte wählen Sie einen Grund."
```

**Notes**
- No-shows feed OBJ-02. Reschedules count as neither cancellation nor no-show in the KPI.
- Marking absence for a whole day uses US-031 (working-hours conflict resolution), which forces each affected appointment to be reassigned or canceled with a reason.

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [Wireframe m-03 Appointment start times](../../03-design/wireframes/m-03-appointment-start-times.svg)
- [Sequence diagrams](../../03-design/diagrams/sequence-diagrams.md)
- [Test cases](../../06-quality/test-cases.md)
