# Post-Incident Review: INC-2026-006 Stale Allergen Information

## Document control

| Field | Value |
|---|---|
| Document ID | PIR-2026-006 |
| Version | 1.1 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Engineering Lead (Incident Commander), backend developer (Technical Lead), Partner Success Lead (Communications Lead), Compliance Lead, Product Owner, QA Lead |

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-08-24 | Approved at the review meeting |
| 1.1 | 2026-09-25 | CAPA status updated; requirement changes baselined in SRS v1.3 (CR-005) |

## Incident summary

| Field | Value |
|---|---|
| Incident ID | INC-2026-006 |
| Title | Outdated allergen information shown after a menu change |
| Severity | SEV-1 (customer-safety near miss; no harm reported) |
| Status | Closed (CAPA-006-03 and CAPA-006-08 ship with R1.1 on 2026-10-07) |
| Incident date | Friday 2026-08-14 |
| Start | 08:52 CEST (first change not shown to customers) |
| Detected | 12:31 (customer email to support) |
| Mitigated | 13:40 |
| Resolved | 17:10 (all affected customers contacted) |
| Incident Commander | Engineering Lead |
| PIR author | Business Analyst |
| Related CR | CR-005 |
| Related requirements | BR-013, FR-CAT-04, FR-ORD-12, NFR-DAT-02, NFR-OBS-02 |

## 1. Executive summary

At 08:52 the Business Admin of Konditorei Pomeranze confirmed a new allergen declaration for "Topfentorte (Stück)". Hazelnut brittle had been added to the recipe, so the declaration changed from A, C, G to A, C, G, H. The portal confirmed the save, but the app kept showing A, C, G for almost five hours. Customer menus were served through the CDN with a 24-hour cache, a setting chosen at launch to meet the cold-start target and never reviewed against allergen data.

At 11:37 a customer with a tree-nut allergy and the allergen filter set to H ordered two slices. She saw no H and no filter marking. At pick-up a staff member mentioned the new topping, and she declined the order and did not eat it.

The team purged the cache and limited menu caching to 30 seconds within 42 minutes of declaring the incident. The impact analysis found 11 orders, from 9 customers, that contained an item whose newly added allergen had not been shown. All 9 were contacted the same afternoon, and none reported a reaction. CR-005 made data freshness a requirement (NFR-DAT-02), added declaration versions to the cart, and added re-confirmation at checkout.

## 2. Customer and business impact

| Dimension | Impact |
|---|---|
| Customers | 9 customers placed 11 orders containing an item whose newly added allergen was not displayed when they ordered. 1 near miss reported (the customer declined at pick-up). No reactions reported after contact. |
| Businesses | 6 businesses had at least one affected order; Konditorei Pomeranze had 4 |
| Declaration changes in the exposure window | 41 confirmed changes on 29 items at 14 businesses between 2026-07-01 and 2026-08-14 13:22; 17 of them added an allergen |
| Orders within 24 h after a change | 187 |
| Money | The near-miss order (EUR 9.20) was refunded by Konditorei Pomeranze; Quartier reversed its commission |
| Compliance | Compliance Lead assessed on 2026-08-14: no personal data breach. The businesses had provided correct information in their own premises; the platform had displayed an outdated version. Businesses informed in writing. |

## 3. Timeline

All times CEST on 2026-08-14 unless stated.

| Time | Event | Actor (role) |
|---|---|---|
| 2026-07-01 | Launch configuration caches `GET /locations/{id}/catalog` at the CDN for 24 hours as "static content" | Release (Engineering) |
| 2026-08-13 21:40 | CDN caches Pomeranze's catalog with "Topfentorte (Stück)" at version 3 (A C G) | System |
| **08:52** | **Start:** Business Admin confirms version 4 (A C G H). The portal shows "Gespeichert". | Business Admin (BIZ-ADM) |
| 11:37 | A customer with filter H orders 2 x Topfentorte (Stück) for 12:00 to 12:15; the app shows A C G and no filter marking | Customer |
| 11:39 | Order accepted; payment captured | Staff (BIZ-STF) |
| 12:04 | At hand-over, staff mention the hazelnut brittle; the customer declines; the order is not handed over | Staff, customer |
| 12:20 | Business Admin refunds EUR 9.20 | Business Admin |
| **12:31** | **Detected:** customer emails support: "The app said no nuts." | Customer |
| 12:44 | Support compares app and portal: A C G in the app, A C G H in the portal; pages on-call | Partner Success |
| **12:58** | **SEV-1 declared** (customer safety). IC: Engineering Lead. BA and Compliance Lead joined. | IC |
| 13:10 | Cause located: catalog responses served from the CDN with `max-age=86400` | Technical Lead |
| 13:22 | CDN purged for all catalog paths; CDN rule for catalog set to 30 seconds | Technical Lead |
| **13:40** | **Mitigated:** freshness verified on 20 sampled businesses | Technical Lead, QA Lead |
| 14:10 | BA impact query (counts and order IDs only): 41 changes, 187 orders within 24 h, 11 orders where an added allergen was not shown | Business Analyst |
| 15:30 | Contact rule agreed: each of the 9 customers emailed with the correct allergens; the near-miss customer also called | Compliance Lead, Product Owner |
| 16:45 | Emails sent; Partner Success calls the 6 businesses | Communications Lead |
| **17:10** | **Resolved:** all customers contacted; caching verified again | IC |
| 2026-08-17 | CR-005 raised | Business Analyst |
| 2026-08-20 | CR-005 approved (emergency) | Product Owner, Engineering Lead |
| 2026-08-21 | Server hotfix: ETag per declaration version; `max-age=30, must-revalidate` in the API; outdated cart lines rejected at placement with a generic message; portal shows "Live in der App in spätestens 1 Minute · Version n" | Engineering |
| 2026-08-24 | PIR review meeting | Business Analyst |
| 2026-09-25 | SRS v1.3 baselined; app re-confirmation UI (US-020) in R1.1 | Business Analyst |

## 4. Detection analysis

Detection took 3 hours 39 minutes from the start, and it came from a customer.
- **No signal existed.** Nothing compared what the API had confirmed with what customers were served. Monitoring watched errors and latency, and the CDN was healthy.
- **The test set-up could not see it.** Staging had no CDN in front of the API, so every allergen test read fresh data.
- **With today's controls** (CAPA-006-04), the freshness probe confirms a change on a probe item every 5 minutes and alerts when the served version lags for more than 60 seconds. The cache rule itself (CAPA-006-01) removes the window.

## 5. Response analysis

| Measure | Target (SEV-1) | Actual | Met? |
|---|---|---|---|
| Acknowledge | 15 min | 13 min (12:31 to 12:44) | Yes |
| Declare | n/a | 14 min after acknowledgment | n/a |
| Mitigate | 2 h | 42 min after declaration | Yes |
| Customer contact | Same day | All 9 by 16:45 | Yes |

The impact analysis needed the change history of declarations, which existed because declarations had been audited from the start (BR-045). Without it, the team would have had to contact every customer who had ordered from the 14 businesses.

## 6. Root cause analysis

### 6.1 Five whys

1. **Why did the customer see A, C, G?** Because the app showed a catalog served by the CDN, which still held version 3.
2. **Why did the CDN hold an outdated version?** Because catalog responses were cached for 24 hours as static content.
3. **Why were they treated as static?** Because the launch performance work for the cold-start target (NFR-PERF-02) cached all public reads under `/locations/*`. It was a technical optimization with no requirement on how fresh the data must be.
4. **Why was there no freshness requirement?** Because BR-013 (v1.2) said every change needs confirmation but not when customers see it. The SRS had no NFR for data freshness, and allergen tests ran against the API origin only.
5. **Why did checkout not catch the difference?** Because the cart did not carry the declaration version the customer saw, and checkout re-checked prices and availability but not allergens.

**Root cause statement:** Allergen data had no freshness requirement, so a performance cache served a superseded declaration for hours, and checkout had no version check that could catch the difference before payment.

### 6.2 Contributing factors

| Category | Factor |
|---|---|
| Requirements | No freshness NFR; BR-013 silent on visibility; no version on cart lines |
| Technology | CDN rule matched all `/locations/*` paths; no ETag tied to declaration versions |
| Process | The caching decision was not reviewed by the BA or the Compliance Lead; staging lacked the CDN |
| User interface | The portal said "Gespeichert" with no indication of when customers would see the change |

### 6.3 Requirement-gap classification

| Cause | Classification | Evidence |
|---|---|---|
| No freshness requirement for allergen data | Requirements gap | SRS v1.2: no NFR on staleness |
| No version check at checkout | Requirements gap | BR-013 and FR-ORD-04 (v1.2) |
| CDN rule cached safety data | Design defect | Launch CDN configuration |
| Tests never went through the CDN | Test gap | Staging topology |

## 7. What went well, what went poorly, where we got lucky

| What went well | What went poorly | Where we got lucky |
|---|---|---|
| The customer reported quickly and precisely | Nearly 4 hours with outdated allergen information | The staff member mentioned the new topping at hand-over |
| Audited declarations made the impact list exact | The portal told the business "saved" without saying when it would be live | It happened in the school summer holidays, when order volume in the pilot districts was about a third lower than in the first two weeks of July |
| Mitigation by configuration in 42 minutes | Staging could not reproduce the problem | Most of the 41 changes in the window removed an allergen or were made before opening |

## 8. Corrective and preventive actions

| ID | Action | Type | Owner (role) | Due | Status | Tracking |
|---|---|---|---|---|---|---|
| CAPA-006-01 | Purge the CDN and limit menu caching to 30 seconds with revalidation | Mitigate | Engineering Lead | 2026-08-14 | Done | DEC-13 |
| CAPA-006-02 | ETag per declaration version; server rejects placement of outdated lines | Prevent | Engineering Lead | 2026-08-21 | Done (hotfix) | CR-005; FR-ORD-12 |
| CAPA-006-03 | App re-confirmation of changed allergens and prices at checkout | Prevent | Product Owner | 2026-10-07 | In progress (R1.1) | US-020-AC1, AC2 |
| CAPA-006-04 | Freshness probe and alert on menu cache age above 60 seconds | Detect | Engineering Lead | 2026-09-04 | Done | NFR-DAT-02, NFR-OBS-02, TC-NFR-012 |
| CAPA-006-05 | Staging puts the CDN in front of the API | Process | Engineering Lead | 2026-09-11 | Done | Test strategy section 4 |
| CAPA-006-06 | Definition of Done item: stated maximum staleness for safety and money data | Process | Business Analyst | 2026-08-28 | Done | [DoR and DoD](../../05-delivery/definition-of-ready-and-done.md) |
| CAPA-006-07 | Portal message "Live in der App in spätestens 1 Minute · Version n" | Prevent | UX Designer | 2026-08-21 | Done | US-029-AC4 |
| CAPA-006-08 | Email to customers with open pre-orders when allergens change | Prevent | Product Owner | 2026-10-07 | In progress (R1.1) | US-029-AC5; TBD-12 |

## 9. Requirement and documentation changes

| Artifact | Change | Baselined in |
|---|---|---|
| BR-013 | Rewritten: declaration versions; visible within 60 seconds; cart lines keep the version; re-confirmation at checkout | SRS v1.3 (CR-005) |
| NFR-DAT-02 | New: freshness of allergen, price and availability data | SRS v1.3 |
| NFR-OBS-02 | Menu cache age above 60 seconds added | SRS v1.3 |
| FR-CAT-04, FR-ORD-12 | Versioning and re-confirmation | SRS v1.3 |
| US-010, US-020, US-029 | US-010-AC6; US-020-AC1 and AC2; US-029-AC4 and AC5 | Stories v1.3 |
| Spec 001 (allergen filter) | Clarification C-11 and SPEC-FR-015: re-evaluate on refresh; warn when the menu may be outdated | Spec v1.2 |
| Incident process | Safety near misses are SEV-1; allergen contact rule | Process v1.1 |

## 10. Lessons learned

1. **Every piece of data a customer relies on for safety needs a stated maximum age**, and that age is a requirement, not a tuning parameter.
2. **"Saved" is not "live".** A confirmation screen must say when the change reaches customers.
3. **Test through the same path customers use.** A cache in production but not in staging hides a whole class of defects.
4. **Audit trails pay off in the first hour.** The exact impact list came from records kept for a different reason.

## Approval

| Role | Decision | Date |
|---|---|---|
| Incident Commander (Engineering Lead) | Approved | 2026-08-24 |
| Product Owner | Approved | 2026-08-24 |
| Compliance Lead | Approved | 2026-08-24 |
| Business Analyst (author) | Prepared | 2026-08-21 |

## Related documents

- [Incident management process](../incident-management-process.md)
- [Change request log (CR-005)](../../05-delivery/change-request-log.md)
- [Business rules table 3.1c](../../02-requirements/business-rules.md)
- [Non-functional requirements](../../02-requirements/non-functional-requirements.md)
- [EP-03 stories (US-020)](../../05-delivery/user-stories/EP-03-ordering-pickup-payments.md)
- [Allergen filter spec](../../08-ai-assisted-ba/specs/001-allergen-filter-on-menus/spec.md)
- [Test cases](../../06-quality/test-cases.md)
