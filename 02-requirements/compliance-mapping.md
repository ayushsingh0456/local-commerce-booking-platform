# Compliance Mapping: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | LC-REQ-CMP |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst (reviewed by the Compliance Lead and external legal counsel) |
| Last updated | 2026-09-25 |
| Reviewers | Compliance Lead; external data protection officer; Finance Lead; Product Owner |

**Purpose and scope.** This document shows how the platform's requirements support the legal and regulatory obligations that apply to an Austrian local commerce platform. Each row names the obligation, who carries it, the requirements that support it and the evidence. It shows how requirements were derived; it is **not legal advice**. External counsel reviewed the interpretation for the pilot.

**Who is obliged.** the platform is an intermediary. The business is the seller of food and the provider of services, and carries the food information, price and invoice obligations. The platform carries the platform, data protection and payment-channel obligations, and builds the product so that the business can meet its own obligations. The "Obligated party" column makes this explicit.

## 1. Food information and allergens

| # | Obligation | Source | Obligated party | Supported by | Evidence |
|---|---|---|---|---|---|
| F1 | Allergen information for non-prepacked food must be provided to the consumer | Regulation (EU) No 1169/2011 Art. 9(1)(c) and Art. 44; Austrian allergen information regulation | Business | BR-010, BR-011, FR-CAT-04, FR-DSC-06, FR-DSC-07 | TC-CAT-002, TC-DSC-004 |
| F2 | For distance sales, mandatory food information must be available before the purchase is concluded | Regulation (EU) No 1169/2011 Art. 14 | Business; the platform provides the channel | Allergens shown above options and never collapsed (FR-DSC-07); publication gate (BR-011); re-confirmation at checkout when a declaration changed (BR-013, FR-ORD-12) | TC-DSC-004, TC-ORD-011 |
| F3 | The information must be accurate and current | Regulation (EU) No 1169/2011 Art. 7 and 8 | Business | Declaration versions confirmed by a Business Admin; live within 60 s (BR-013, NFR-DAT-02); audit trail (BR-045) | TC-CAT-003, TC-NFR-012 |
| F4 | The 14 allergens and their names | Regulation (EU) No 1169/2011 Annex II | Business | BR-010 with the letter codes customary in Austria | TC-DSC-004 |

## 2. Data protection

| # | Obligation | Source | Obligated party | Supported by | Evidence |
|---|---|---|---|---|---|
| D1 | Data minimization and purpose limitation | GDPR Art. 5(1)(c) | The platform (controller for customer accounts) | Sign-up data set limited to BR-002; no address or date of birth | TC-ACC-002 |
| D2 | Consent of a child to an information society service is valid from 14 in Austria | GDPR Art. 8; DSG section 4(4) | The platform | Self-declaration "I am at least 14 years old" (BR-002). It is not verified; the risk was accepted by the Compliance Lead because no service is age-restricted | TC-ACC-002 |
| D3 | Consent must be freely given, specific, separate and as easy to withdraw as to give | GDPR Art. 7 | The platform | Separate unticked opt-ins; consent records with version; withdrawal in Profile (BR-003, FR-ACC-08) | TC-ACC-006 |
| D4 | Storing or reading information on the device (analytics SDK) needs consent | TKG 2021 section 165 | The platform | Analytics starts only after opt-in (FR-ACC-08) | TC-ACC-006 |
| D5 | Health data (special category) | GDPR Art. 9 | The platform | The allergen filter stays on the device and is never sent to the server, analytics or error monitoring (FR-DSC-09, NFR-OBS-01). Saving it to the account would need explicit consent (TBD-07) | TC-DSC-008, TC-NFR-009 |
| D6 | Right to erasure and right of access and portability | GDPR Art. 15, 17, 20 | The platform | Account deletion within 30 days; pseudonymized retention of records (BR-005, FR-ACC-07); JSON export (FR-ACC-09) | TC-ACC-007, TC-ACC-008 |
| D7 | Processors under contract; data in the EU | GDPR Art. 28 and Chapter V | The platform | EU hosting; processor register (NFR-PRIV-01) | Processor register review |
| D8 | Security of processing | GDPR Art. 32 | The platform and the business | NFR-SEC-01 to NFR-SEC-05; business isolation by row-level security | TC-NFR-005, TC-NFR-006 |
| D9 | Customer data the business receives (first name, phone for a specific order) | GDPR Art. 6(1)(b) | Business (controller for its own customer relationship) | Order ticket shows first name and slot only (FR-POS-09); phone shown only in order detail for contact about that order | TC-POS-006 |

## 3. Payments

| # | Obligation | Source | Obligated party | Supported by | Evidence |
|---|---|---|---|---|---|
| P1 | Strong customer authentication for online card payments | PSD2 Art. 97; Commission Delegated Regulation (EU) 2018/389; ZaDiG 2018 | Card payment provider and issuer; the platform must not prevent it | 3-D Secure 2 through hosted components (BR-032); holds survive a challenge (BR-019, BR-020) | TC-PAY-001, TC-ORD-010 |
| P2 | Card data security | PCI DSS v4.0 | The platform (SAQ A scope) | Hosted fields only (NFR-SEC-02) | SAQ A attestation |
| P3 | The platform must not hold customer funds without a payment license | PSD2 / ZaDiG 2018 scope | The platform | Connected accounts: funds flow from the provider to the business; commission taken as an application fee ([ADR-003](../03-design/architecture/adr/ADR-003-marketplace-payments-authorize-capture.md)) | Architecture review |
| P4 | Business identity and bank checks before payouts | Anti-money-laundering rules applied by the provider | Card payment provider | Hosted onboarding (FR-ONB-03); go-live gate L2 (BR-043) | TC-ONB-003 |

## 4. VAT, invoices and records

| # | Obligation | Source | Obligated party | Supported by | Evidence |
|---|---|---|---|---|---|
| T1 | Consumer prices shown including VAT | Price marking act (PrAG); UStG | Business | All prices gross (BR-015) | TC-DSC-003 |
| T2 | Correct VAT rate per supply: 10% food, 20% standard (most drinks, services) | UStG section 10 | Business | Tax category per item, choice and service; effective-dated rates (BR-015, FR-ONB-06) | TC-ONB-006 |
| T3 | Invoice content; small-amount invoice up to EUR 400 gross | UStG section 11, including 11(6) | Business | Receipt in the business's name with per-rate VAT (BR-038, FR-PAY-07); in-app limit EUR 400.00 | TC-PAY-006 |
| T4 | Discounts reduce the taxable amount per supply | UStG section 16 | Business | Pro rata allocation per rate group (BR-035) | TC-PAY-003 |
| T5 | Retention of records for 7 years | BAO section 132 | Business; the platform for its own commission invoices | Orders, payments and receipts kept 7 years, pseudonymized after deletion (BR-005) | TC-ACC-007 |
| T6 | Cash register obligation for cash sales, including card payments at the counter | BAO section 131b; cash register security regulation (RKSV) | Business | The platform takes no counter payments (CR-003 rejected); "Bezahlt an der Kassa" is a status only (BR-024). Whether prepaid app orders also need a register receipt is open (TBD-01) | TC-POS-004 |
| T7 | VAT on the platform's commission | UStG | The platform | 20% VAT on the commission; monthly commission invoice (BR-037) | TC-PAY-008 |

## 5. Consumer protection and e-commerce

| # | Obligation | Source | Obligated party | Supported by | Evidence |
|---|---|---|---|---|---|
| C1 | Order button must state the obligation to pay | FAGG section 8(2) | Business; the platform builds the button | "Zahlungspflichtig bestellen" and "Zahlungspflichtig buchen" (FR-ORD-04, FR-APT-09) | TC-ORD-004, TC-APT-004 |
| C2 | Pre-contract information: identity of the seller, total price | FAGG section 4; ECG section 5 | Business | Legal information on the business page (FR-DSC-05); seller shown at checkout | TC-DSC-005 |
| C3 | Right of withdrawal exceptions: perishable goods; services for a specific date | FAGG section 18(1) | Business | Interpretation confirmed by external counsel for the pilot; customer cancellation rules are contractual (BR-023, BR-029) | Legal opinion on file |
| C4 | Reviews: if published, state whether and how authenticity is checked | Directive (EU) 2019/2161 (Omnibus); UWG | The platform | No public reviews (BR-008; CR-007 rejected) | TC-DSC-002 |

## 6. Platform-to-business and digital services

| # | Obligation | Source | Obligated party | Supported by | Evidence |
|---|---|---|---|---|---|
| B1 | Clear terms for business users; notice before changes | Regulation (EU) 2019/1150 Art. 3 | The platform | Business terms version accepted at application (BR-042) | TC-ONB-001 |
| B2 | Statement of reasons for suspension; 30 days' notice for termination | Regulation (EU) 2019/1150 Art. 4 | The platform | BR-044, FR-ONB-05 | TC-ONB-005 |
| B3 | Disclosure of the main ranking parameters | Regulation (EU) 2019/1150 Art. 5 | The platform | Ranking by distance or name only, disclosed in the terms (BR-006, BR-008) | TC-DSC-001 |
| B4 | Traceability of traders on online marketplaces | Regulation (EU) 2022/2065 (DSA) Art. 30 | Not applicable while the platform is a micro or small enterprise (Art. 29) | The onboarding data set already covers it, except an identity document copy, which the card payment provider holds (BR-042, TBD-10) | Gap note in TBD-10 |

## 7. Accessibility

| # | Obligation | Source | Obligated party | Supported by | Evidence |
|---|---|---|---|---|---|
| A1 | Accessibility of e-commerce services | Directive (EU) 2019/882; Austrian accessibility act (BaFG) | The platform, once it is no longer a micro-enterprise | WCAG 2.2 AA and the mobile criteria in NFR-ACC-01 | Accessibility audit; TC-NFR-008 |

## 8. Open compliance items

| Item | Status | Owner |
|---|---|---|
| Register receipt for prepaid app orders (TBD-01) | Written opinion requested from tax counsel | Compliance Lead |
| Explicit-consent design if the allergen filter is ever saved to the account (TBD-07) | Not planned for R1 | Compliance Lead |
| DSA and accessibility obligations at growth thresholds (TBD-10) | Monitored each quarter against headcount and turnover | Compliance Lead |

## Related documents

- [Business Requirements Document](BRD.md)
- [Software Requirements Specification](SRS.md)
- [Business rules](business-rules.md)
- [Non-functional requirements](non-functional-requirements.md)
- [Glossary](glossary.md)
- [ADR-003 Marketplace payments](../03-design/architecture/adr/ADR-003-marketplace-payments-authorize-capture.md)
- [Test cases](../06-quality/test-cases.md)
