# Glossary: Quartier

## Document control

| Field | Value |
|---|---|
| Document ID | QT-REQ-GLO |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Product Owner; UX Designer; Compliance Lead |

**Purpose and scope.** Shared vocabulary for requirements, design, tests and the user interface. Where the product uses a German term in the UI, the German term is given first, with its English equivalent. Terms are listed alphabetically by the term used in the requirements.

## Terms

| Term | Definition | Used in |
|---|---|---|
| Acceptance timeout | The 5 minutes a business has to accept or reject an order before it is auto-rejected (BR-022). | FR-ORD-07 |
| Allergen declaration | The set of allergens confirmed for an item or option choice, or the explicit confirmation that it contains none of the 14. Versioned; each change needs confirmation (BR-011, BR-013). | FR-CAT-04 |
| Allergen filter | Customer setting, stored only on the device, that marks menu items containing chosen allergens (FR-DSC-09). | Spec 001 |
| Allergen letter code | Letter used in Austria for each of the 14 allergens: A, B, C, D, E, F, G, H, L, M, N, O, P, R (BR-010). | FR-DSC-06 |
| Authorisation | A reservation of the amount on the customer's card; nothing is charged until capture (BR-033). | FR-PAY-02 |
| BAO | Bundesabgabenordnung, the Austrian Federal Fiscal Code: record retention (section 132) and the cash register obligation (section 131b). | Compliance mapping |
| Beliebig (any available) | Appointment option in which the customer does not choose a staff member; the system assigns one (BR-027). | FR-APT-02 |
| Bezahlt an der Kassa (paid at the counter) | Status set by staff when a pay-at-pickup order was paid on the business's own register (BR-024). | FR-PAY-03 |
| Booking | Umbrella term in the customer app for an order or an appointment; the Bookings tab lists both. | FR-ORD-09 |
| Buffer | Time after a service in which the staff member is not bookable, for cleaning or preparation (BR-026). | FR-CAT-06 |
| Business | A legal entity on Quartier (B-nnnn) with one or more locations. | All |
| Capture | Charging an authorized amount; done when the business accepts (BR-033). | FR-PAY-02 |
| Card payment provider (PSP) | Licensed third party that processes card and wallet payments, onboards businesses and pays them out (IF-01). | ADR-003 |
| Commission | Quartier's fee: 8% of the paid gross after business-funded discounts, plus 20% VAT (BR-037). | FR-PAY-08 |
| Connected account | The business's account at the card payment provider, into which customer payments are settled. | ADR-003 |
| Cutoff | The latest time a customer can cancel in the app: 30 minutes before the pickup slot for orders, 24 hours before the start for appointments (BR-023, BR-029). | FR-ORD-08 |
| Declaration version | Sequential number of an allergen declaration; a cart line keeps the version the customer saw. | BR-013 |
| FAGG | Fern- und Auswärtsgeschäfte-Gesetz, the Austrian distance and off-premises selling act. | FR-ORD-04 |
| Firmenbuchnummer | Number of a company in the Austrian commercial register, format "FN 123456a". | FR-ONB-01 |
| GISA-Zahl | Number in the Austrian trade license register (Gewerbeinformationssystem Austria). | FR-ONB-01 |
| Go-live gate | The conditions that must all hold before a business can be published (BR-043). | FR-ONB-04 |
| Hold | A short reservation of slot capacity or of a staff member's time while the customer completes checkout or booking, kept in Redis with a time-to-live (BR-019, BR-028). | ADR-002 |
| Kleinunternehmer (small business) | A business that applies the Austrian small-business VAT exemption; its items use tax category Exempt (BR-015). | FR-CAT-03 |
| Lead time (Vorlaufzeit) | Minimum time between ordering and pickup: the location's preparation lead time (default 20 minutes) or an item's pre-order lead time (BR-017). | FR-ORD-02 |
| Live | Business status in which it appears in discovery and can take orders or appointments. | BR-006 |
| Location (Standort) | A physical site of a business (L-nnnn) with its own address, hours, pickup settings and staff. | FR-CAT-02 |
| Minimum notice | The earliest an online appointment can start after now: default 2 hours (BR-026). | FR-APT-02 |
| No-show | Appointment status set by staff when the customer did not come; allowed from 15 minutes after the start (BR-030). | FR-POS-06 |
| Not collected (nicht abgeholt) | Order status set by staff when a Ready order was not picked up; allowed from 60 minutes after the slot ends (BR-024). | FR-POS-08 |
| Occupied interval | The time an appointment blocks a staff member: start to start plus duration plus buffer. | BR-026 |
| Order board | The POS view screen on which staff see and progress pickup orders in real time. | FR-POS-01 |
| Pause | Temporary stop of new orders for a location, set by staff or automatically (BR-040). | FR-POS-05 |
| Pickup code (Abholcode) | Four-character code, also shown as a QR code, that staff match before handing over an order (BR-024). | FR-ORD-05 |
| Pickup slot (Abholzeit) | A 15-minute window in which the customer collects an order (BR-017). | FR-ORD-02 |
| POS view | The restricted Business Web Portal view for Staff: order board, sold-out toggles, pause and the appointment day view. It is not a cash register. | FR-POS-07 |
| Pre-order (Vorbestellung) | An order containing an item with a lead time, for a later day, up to 7 days ahead (CR-001). | FR-ORD-11 |
| RKSV | Registrierkassensicherheitsverordnung, the Austrian cash register security regulation. Quartier is not a cash register (CR-003). | Compliance mapping |
| SCA / 3-D Secure | Strong customer authentication under PSD2; for cards, the 3-D Secure 2 protocol, sometimes with a challenge in the banking app. | FR-PAY-01 |
| Slot capacity | Maximum number of orders per pickup slot and location (BR-018). | FR-CAT-02 |
| Sold out (ausverkauft) | Item state set by staff; the item cannot be added (BR-014). | FR-POS-04 |
| Statement of reasons | Written explanation sent to a business when Quartier suspends it (BR-044). | FR-ONB-05 |
| Tax category (Steuerkategorie) | Reduced 10%, Standard 20% or Exempt 0% (BR-015). | FR-CAT-03 |
| UID-Nummer | Austrian VAT identification number, format "ATU" plus 8 digits. | FR-ONB-01 |
| Verification | Platform Admin check of a business application against the registers and the UID service. | FR-ONB-02 |
| VIES | The EU's VAT number validation service (IF-07). | FR-ONB-02 |
| Voucher (Gutschein) | Promotional code, funded by the business or by Quartier, that reduces an order total (BR-036). It is not a stored-value gift card. | FR-PAY-06 |

## Acronyms

| Acronym | Expansion |
|---|---|
| ADR | Architecture decision record |
| API | Application programming interface |
| BR | Business rule |
| BRD | Business requirements document |
| CAPA | Corrective and preventive action |
| CR | Change request |
| DSA | Digital Services Act, Regulation (EU) 2022/2065 |
| DSG | Datenschutzgesetz, the Austrian Data Protection Act |
| ECG | E-Commerce-Gesetz, the Austrian E-Commerce Act |
| FCM | Firebase Cloud Messaging |
| FR | Functional requirement |
| GDPR | General Data Protection Regulation, Regulation (EU) 2016/679 |
| KYC | Know your customer: identity checks on a business by the card payment provider |
| NFR | Non-functional requirement |
| P2B | Platform-to-business regulation, Regulation (EU) 2019/1150 |
| PIR | Post-incident review |
| PSD2 | Revised Payment Services Directive, Directive (EU) 2015/2366 |
| RLS | Row-level security (PostgreSQL) |
| RTM | Requirements traceability matrix |
| SRS | Software requirements specification |
| UAT | User acceptance testing |
| UStG | Umsatzsteuergesetz, the Austrian VAT Act |
| VAT | Value added tax (Umsatzsteuer, USt.) |

## Related documents

- [Software Requirements Specification](SRS.md)
- [Business rules](business-rules.md)
- [Compliance mapping](compliance-mapping.md)
- [Data dictionary](../03-design/data/data-dictionary.md)
