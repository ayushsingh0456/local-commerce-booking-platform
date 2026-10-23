# Synthetic Test Data: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | LC-QA-04 |
| Version | 1.3 |
| Status | Approved |
| Owner | QA Lead (with the Business Analyst) |
| Last updated | 2026-09-25 |
| Reviewers | Engineering Lead; Compliance Lead |

**Purpose and scope.** The synthetic data used by the test cases, the stories, the wireframes and UAT, so that every artifact describes the same businesses, items and appointments. All names, addresses, register numbers, UID numbers and phone numbers are invented. Addresses use real Vienna postcodes with synthetic street names and numbers. Nothing here comes from a real business or customer.

## 1. Files

| File | Rows | Content | Used by |
|---|---|---|---|
| [menu-items.csv](menu-items.csv) | Items and option choices | Prices including VAT, tax categories, allergen letters, declaration versions, lead times, visibility | TC-DSC-*, TC-ORD-*, TC-PAY-*, TC-CAT-*, TC-POS-*; US-009 to US-020; wireframes m-01, m-02, w-01, w-02 |
| [salon-schedule.csv](salon-schedule.csv) | Shifts, appointments, services, settings, expected results | Haarstudio Linde on 2026-10-13 and 2026-10-14 | TC-APT-*, TC-CAT-006, TC-CAT-007, TC-POS-010; US-021 to US-025, US-031; wireframe m-03 |

## 2. Businesses and locations

| Business | Location | Category | Address | Key settings |
|---|---|---|---|---|
| B-1001 Café Mohnblume (Mohnblume Kaffeehaus e.U., FN 512376k, ATU71234567) | L-1001 | FOOD: Café, Bakery | Kornblumengasse 9, 1070 Wien | Open 07:30 to 18:00; ordering 07:30 to 17:30; lead time 20 min; capacity 6; cutoff 30 min; pay at pickup on; phone +43 1 5234567 |
| B-1002 Konditorei Pomeranze | L-1002 | FOOD: Bakery and pastry | Quittengasse 4, 1060 Wien | Open 06:30 to 18:30; pre-orders up to 7 days |
| B-1003 Nudelbar Sesam | L-1003 Mariahilf, L-1004 Neubau | FOOD: Restaurant | Hollerstiege 2, 1060 Wien; Zieglerhof 11, 1070 Wien | Pay at pickup off; voucher SESAM5 |
| B-1004 Kaffeebar Zimtstern | L-1005 | FOOD: Café | Kardamomgasse 3, 1070 Wien | Small-business VAT exemption; tax category Exempt |
| B-2001 Haarstudio Linde (Haarstudio Linde Leitner GmbH, FN 498812t, ATU69876543) | L-2001 | SALON: Hair salon | Lindenhofgasse 17, 1070 Wien | Notice 2 h; horizon 8 weeks; free cancellation 24 h; phone +43 1 5267788 |
| B-3001 Massageatelier Ruhepol | L-3001 | SPA: Massage studio | Mondscheinweg 6, 1060 Wien | Used for the go-live gate row L6 |

## 3. People and accounts

| Account | Person | Role | Notes |
|---|---|---|---|
| U-CUS-0001 | Sophie Wallner, sophie.wallner@example.com, +43 664 1234567 | Customer (PER-01) | German UI; iPhone |
| U-CUS-0002 | Daniela Costa, daniela.costa@example.com, +43 676 5550123 | Customer (PER-02) | English UI; Android; tree-nut allergy (filter H) |
| U-BIZ-1001 | Elif Aydin, elif.aydin@mohnblume.example | Business Admin, B-1001 (PER-03) | Portal account; email reserved for portal (BR-002) |
| U-BIZ-1002 | Tobias Gruber | Staff, L-1001 (PER-04) | POS tablet |
| U-BIZ-2001 | Martina Leitner | Business Admin, B-2001 (PER-05) | |
| U-BIZ-2002, U-BIZ-2003 | Lena Auer, Aylin Demir | Staff, bookable, L-2001 | Lena: Damenhaarschnitt, Herrenhaarschnitt; Aylin: also Ansatzfarbe |
| U-PLT-0001 | Paul Haider | Platform Admin (PER-06) | TOTP in staging |

## 4. Vouchers

| Code | Funded by | Value | Minimum | Validity | Limits |
|---|---|---|---|---|---|
| MOHN3 | Café Mohnblume | EUR 3.00 | EUR 15.00 | 2026-09-01 to 2026-12-31 | Once per customer; 200 total |
| SOMMER25 | Café Mohnblume | 25% | none | 2026-07-01 to 2026-08-31 | Expired (row G2) |
| SESAM5 | Nudelbar Sesam | EUR 5.00 | EUR 20.00 | 2026-09-01 to 2026-12-31 | Other business (row G4) |
| WILLKOMMEN3 | The platform | EUR 3.00 | EUR 15.00 | 2026-09-01 to 2026-12-31 | Once per customer |

## 5. Expected results for the worked examples

**VAT basket (TC-PAY-003, US-014, SRS Appendix B.1).** IT-1001 x 2, IT-1002 x 1, IT-1003 with CH-1003-2 x 1, voucher MOHN3:

| VAT rate | Gross before discount | Discount | Gross | Net | VAT |
|---|---|---|---|---|---|
| 10% | 13.70 | 2.20 | 11.50 | 10.45 | 1.05 |
| 20% | 5.00 | 0.80 | 4.20 | 3.50 | 0.70 |
| Total | 18.70 | 3.00 | 15.70 | 13.95 | 1.75 |

Commission 1.26, VAT on commission 0.25, payout 14.19 (TC-PAY-008).

**Salon start times (TC-APT-002, US-022, SRS Appendix B.2).** Rows SS-01, SS-03 and SS-05; expected list in SS-08: 09:00, 09:15, 09:30, 11:15, 11:30, 11:45, 12:00. After booking 09:30: 11:15, 11:30, 11:45, 12:00.

## 6. Conventions and reset

- Prices in the CSV are EUR with a decimal point; the app shows them as "€ 3,40".
- Allergen letters are space-separated in BR-010 order; an empty value with `contains_none_confirmed = true` means "contains none of the 14".
- Each test location has a virtual clock in staging. Reset restores this data set and sets the clock to 2026-09-17 11:00 for food locations and 2026-10-12 18:00 for L-2001.

## Related documents

- [Test cases](../test-cases.md)
- [Test strategy and plan](../test-strategy-and-plan.md)
- [UAT plan and scripts](../uat-plan-and-scripts.md)
- [Business rules](../../02-requirements/business-rules.md)
