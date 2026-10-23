# Personas: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DIS-PER |
| Version | 1.1 |
| Status | Approved |
| Owner | Business Analyst (with the UX Designer) |
| Last updated | 2026-04-15 |
| Reviewers | Product Owner; Partner Success Lead |

**Purpose and scope.** Six personas built from the discovery interviews, lunch observations and the resident survey. They are composites; no persona describes a real person. Each persona lists the requirements it drives, so design decisions can be traced back to a person's need.

## PER-01 Sophie Wallner, customer (CUS)

| Attribute | Detail |
|---|---|
| Profile | 31, UX writer at an agency in Neubau; iPhone; German |
| Context | Orders lunch for herself and two colleagues three times a week; books a haircut every 6 weeks |
| Goals | Order on the way down the stairs; collect without queueing; book a haircut in the evening |
| Frustrations | "I call at 11:50 and it is engaged." "The salon only answers between clients." |
| Quote | "If I have to ring, I go to the supermarket instead." |
| Drives | NFR-USE-01 (two taps), BR-017 (pickup slots), BR-026 (start times that fit), FR-APT-05 (reminders) |

## PER-02 Daniela Costa, customer with an allergy (CUS)

| Attribute | Detail |
|---|---|
| Profile | 34, born in Porto, living in Mariahilf for 6 years; data analyst; Android; English UI; tree-nut allergy |
| Context | Asks about nuts at every counter; avoids bakeries she does not know |
| Goals | See allergens before choosing; trust that what she sees is current; keep her allergy private |
| Frustrations | "The letters on the board are from last year." "Staff say 'should be fine'." |
| Quote | "I don't need the app to tell me it's safe. I need it to tell me what's in it." |
| Drives | BR-010 to BR-013, FR-DSC-09 (allergen filter on the device), NFR-I18N-01, NFR-DAT-02 |

## PER-03 Elif Aydin, café owner (BIZ-ADM)

| Attribute | Detail |
|---|---|
| Profile | 42, owner of Café Mohnblume (café with a bakery counter, 6 staff) |
| Context | Maintains the menu at night; handles VAT with a tax adviser; has a certified cash register |
| Goals | Fewer phone orders at lunch; prepaid orders that get collected; money paid out reliably |
| Frustrations | Commission on delivery platforms; tools that take a week to set up |
| Quote | "I'll put my allergens in once, properly. Don't make me do it in three places." |
| Drives | FR-CAT-03 to FR-CAT-05, BR-011, BR-035 to BR-038, OBJ-05 |

## PER-04 Tobias Gruber, counter staff (BIZ-STF)

| Attribute | Detail |
|---|---|
| Profile | 24, student, works the lunch shift at Café Mohnblume |
| Context | Noisy counter, hands busy, a queue in front of him; shared tablet |
| Goals | See new orders without looking for them; mark sold-out items fast; hand over the right bag |
| Frustrations | Phone orders with misheard names; customers waiting for orders that were never written down |
| Quote | "If it beeps once and stops, I'll miss it." |
| Drives | BR-040 (repeating alert, auto-pause), BR-024 (pickup code), FR-POS-04, FR-POS-07 |

## PER-05 Martina Leitner, salon owner (BIZ-ADM)

| Attribute | Detail |
|---|---|
| Profile | 47, owns Haarstudio Linde (3 stylists) |
| Context | Paper diary at the desk; stylists answer the phone between clients |
| Goals | A full diary without the phone; fewer no-shows; never a customer without a stylist |
| Frustrations | 9 to 10% no-shows; double entries in the diary |
| Quote | "Cleaning time is part of the appointment, not a gap." |
| Drives | BR-026 (buffer inside the shift), BR-031 (schedule conflicts), FR-APT-05, CR-004 (raised, rejected) |

## PER-06 Paul Haider, partner operations (PLT-ADM)

| Attribute | Detail |
|---|---|
| Profile | 29, the operator partner operations |
| Context | Verifies 3 to 8 businesses a day; answers business questions |
| Goals | Evidence on one screen; clear go-live checklist; defensible suspensions |
| Frustrations | Chasing documents by email |
| Quote | "Give me the register number, the UID check and a checklist, and I can say yes in 10 minutes." |
| Drives | FR-ONB-02, FR-ONB-04, BR-043, BR-044, BR-045 |

## Design implications across personas

| Implication | Personas | Requirements |
|---|---|---|
| Information before commitment: allergens and total price before any button | PER-01, PER-02 | FR-DSC-07, FR-ORD-04 |
| Say what changed, never silently | PER-02, PER-03 | BR-013, FR-ORD-12 |
| Alerts that cannot be missed in a noisy room | PER-04 | BR-040 |
| Capacity is a promise | PER-01, PER-05 | BR-018, BR-020, BR-026, BR-028 |
| The same numbers everywhere: app, receipt, statement | PER-01, PER-03 | BR-035, BR-037, BR-038 |

## Related documents

- [Stakeholder register and RACI](stakeholder-register-raci.md)
- [Current vs future state](current-vs-future-state.md)
- [User stories](../05-delivery/epics.md)
- [Wireframes](../03-design/wireframes/README.md)
