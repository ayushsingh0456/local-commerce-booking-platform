# Non-Functional Requirements: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | LC-REQ-NFR |
| Version | 1.3 (aligned with SRS v1.3) |
| Status | Approved (baselined) |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Engineering Lead; QA Lead; UX Designer; Compliance Lead |

**Purpose and scope.** This document specifies the 27 quality requirements for the platform R1 and R1.1, classified by the ISO/IEC 25010 product quality model. Every NFR has a measurable target and a verification method. The functional requirements are in the [SRS](SRS.md).

## 1. How to read this document

- **ID** `NFR-<CAT>-NN`. The category is a short code (PERF, CAP, AVL, SEC, PRIV, USE, ACC, I18N, CMP, MOB, DAT, MNT, OBS).
- **Characteristic** is the ISO/IEC 25010 characteristic and sub-characteristic the requirement belongs to.
- **Measure / target** is the pass condition. "p95" means the 95th percentile over the test or measurement window.
- **Verification** is how QA or operations proves it. Release gates are in the [test strategy](../06-quality/test-strategy-and-plan.md).

## 2. Load profile behind the targets

The targets come from discovery and the pilot, not from generic benchmarks.

| Driver | Value | Source |
|---|---|---|
| Lunch peak | 11:45 to 12:45; 41% of daily food orders in the pilot fell into this hour | Pilot order data, July to September 2026 |
| Busiest slot at one location | 6 orders per 15 minutes, the default capacity | Café and restaurant owners in discovery |
| Vienna launch target | 1,500 Live locations and 60,000 monthly active customers by the end of 2027 | Business plan (illustrative) |
| 3-D Secure challenge time | Median 48 s; p90 3 min 40 s; during the issuer slowdown of INC-2026-011, median 9 min and longest 14 min | Pilot payment events; [INC-2026-011](../07-operations/incidents/INC-2026-011-pickup-slots-over-capacity.md) |
| Devices | 58% iOS, 42% Android; median Android device about 3 years old | App store data from the pilot |

## 3. Requirements

| ID | Characteristic (ISO/IEC 25010) | Requirement | Measure / target | Verification | Test cases |
|---|---|---|---|---|---|
| NFR-PERF-01 | Performance efficiency - time behavior | Customer API read operations respond quickly under lunch-peak load. | p95 at or below 300 ms and p99 at or below 800 ms at 150 requests per second | Load test (k6) on staging with production-sized data | TC-NFR-001 |
| NFR-PERF-02 | Performance efficiency - time behavior | Home shows the Near you list quickly on a mid-range Android phone. | Cold start to first business card at or below 3.0 s p90 on a reference device over a 10 Mbit/s connection | Device lab test, 20 runs | TC-NFR-002 |
| NFR-PERF-03 | Performance efficiency - time behavior | Availability for one service, one day and all qualified staff is computed quickly. | p95 at or below 400 ms for a location with 8 staff and 60 appointments that day | Load test and API timing in staging | TC-NFR-003 |
| NFR-PERF-04 | Performance efficiency - time behavior | New orders reach the order board quickly. | p95 at or below 3 s from placement to ticket shown; p99 at or below 6 s | Synthetic order probe every 5 minutes in production | TC-NFR-004 |
| NFR-CAP-01 | Performance efficiency - capacity | The platform carries the Vienna launch volume with headroom. | 1,500 Live locations, 60,000 monthly active customers, 900 orders per 15-minute lunch window and 1,200 concurrent order board connections | Load test before Vienna launch | TC-NFR-001 |
| NFR-AVL-01 | Reliability - availability | Ordering, booking and the order board are available. | 99.5% monthly between 06:00 and 23:00 Europe/Vienna, excluding announced maintenance outside 06:00-23:00 | Uptime monitoring report | TC-NFR-014 |
| NFR-AVL-02 | Reliability - recoverability | Data can be restored after a failure. | RPO 15 minutes, RTO 4 hours; restore rehearsed quarterly | Restore drill record | TC-NFR-014 |
| NFR-AVL-03 | Reliability - fault tolerance | The order board keeps working when the live connection drops. | Falls back to polling every 15 s within 10 s of disconnect and shows "Live-Verbindung unterbrochen"; no order lost | Chaos test: drop WebSocket during lunch simulation | TC-NFR-007 |
| NFR-SEC-01 | Security - integrity | The application is free of serious known weaknesses at each release. | OWASP ASVS 4.0.3 Level 2 controls in scope; no open Critical or High finding from the release penetration test | External penetration test; SAST and dependency scan in CI | TC-NFR-005 |
| NFR-SEC-02 | Security - confidentiality | Card data never reaches the platform systems. | Payment entry only through the provider's hosted fields or wallet sheets; PCI DSS self-assessment SAQ A scope | Architecture review; annual SAQ A attestation | TC-NFR-015 |
| NFR-SEC-03 | Security - authenticity | Credentials and sessions are protected. | Passwords hashed with Argon2id; access tokens 15 minutes; refresh tokens 30 days, rotated on use and revoked on password change | Code review; security test cases | TC-NFR-005 |
| NFR-SEC-04 | Security - authenticity | Portal accounts use a second factor. | TOTP second factor mandatory for Business Admin and Platform Admin; optional for Staff on shared POS tablets with a 12-hour session | Security test cases | TC-NFR-005 |
| NFR-SEC-05 | Security - accountability | Business data is isolated per business. | Every business-scoped table carries business_id, enforced by PostgreSQL row-level security; cross-business access attempts return 404 and are logged | Automated isolation test suite | TC-NFR-006 |
| NFR-PRIV-01 | Security - confidentiality (privacy) | Personal data is minimized and stays in the EU. | Sign-up collects only the BR-002 fields; all processing and storage in EU regions; every processor under a GDPR Article 28 agreement | Data protection impact assessment; processor register review | TC-NFR-009, TC-NFR-018 |
| NFR-PRIV-02 | Security - confidentiality (privacy) | Data subject requests are met on time. | Erasure within 30 days (BR-005); access export within 30 days (FR-ACC-09); request log kept | Quarterly audit of the request log | TC-NFR-018 |
| NFR-USE-01 | Usability - operability | Discovery is fast. | At most 2 taps from app launch to any business page | Usability test with 8 participants; task analysis | TC-NFR-008 |
| NFR-USE-02 | Usability - user error protection | Every validation message states the problem and the fix in the user's language. | German (de-AT) and English text for every message; no raw codes shown | Copy review checklist; UI test assertions | TC-NFR-010 |
| NFR-ACC-01 | Usability - accessibility | Apps are usable with assistive technology. | WCAG 2.2 AA for the web portals; the customer app supports 200% text size, screen readers and 44 x 44 pt targets; open state, status and allergens never rely on color alone | Accessibility audit (EN 301 549 checklist) before Vienna launch | TC-NFR-008 |
| NFR-I18N-01 | Usability - learnability (localization) | The product is German-first. | de-AT default, English selectable; dates DD.MM.YYYY, times HH:mm 24-hour, amounts "€ 1.234,56", distances "2,4 km"; the customer app uses "du", the portals use "Sie" | Localisation test cases on both languages | TC-NFR-010 |
| NFR-CMP-01 | Compatibility - co-existence | Supported platforms. | iOS 16 or later; Android 9 (API 28) or later; current and previous major versions of Chrome, Edge, Firefox and Safari; the POS view on tablets 10 inches or larger in landscape | Device and browser matrix in the test plan | TC-NFR-016 |
| NFR-MOB-01 | Performance efficiency - resource utilization | The customer app is light and stable. | Download size at or below 40 MB; crash-free sessions at or above 99.5% over 7 days | Store metrics; error monitoring dashboard | TC-NFR-002 |
| NFR-DAT-01 | Functional suitability - correctness | Time is handled consistently. | Timestamps stored in UTC and shown in Europe/Vienna; slot and availability grids remain correct on daylight-saving change days (no duplicated or missing 15-minute slot) | Boundary test cases on 2026-10-25 and 2027-03-28 | TC-NFR-011 |
| NFR-DAT-02 | Functional suitability - correctness | Allergen, price and availability data shown to customers is fresh. | A confirmed change is visible in every customer channel within 60 s; menu responses are never cached for more than 60 s by any shared cache | Automated freshness probe; cache header test | TC-NFR-012 |
| NFR-MNT-01 | Maintainability - testability | Business rules are protected by automated tests. | Every BR has at least one automated test that fails when the rule is broken; CI blocks a merge that lowers rule coverage | CI coverage gate report | TC-NFR-017 |
| NFR-MNT-02 | Maintainability - modifiability | Regulated values change without a release. | VAT rates, slot defaults, cancellation cutoffs and notice periods are configuration or effective-dated reference data | Configuration review; test case on a rate change | TC-NFR-017 |
| NFR-OBS-01 | Maintainability - analyzability | Errors are visible without leaking personal data. | All apps report to error monitoring with release and device context; personal data and allergen selections are scrubbed before sending | Error event sample review each release | TC-NFR-009 |
| NFR-OBS-02 | Reliability - maturity | Business anomalies alert on-call staff. | Alerts on: any slot over capacity; 3-D Secure challenge duration p90 above 5 minutes; slot holds expiring during payment above 5% in 15 minutes; menu cache age above 60 s; order board disconnected while ordering is open | Alert test in staging each release | TC-NFR-013 |

## 4. Notes by characteristic

**Performance and capacity.** NFR-PERF-04 is the one the businesses feel. An order that reaches the board late eats into the 5-minute acceptance window (BR-022). The synthetic probe places a real 1-cent test order at a hidden test location every 5 minutes and measures time to board. NFR-CAP-01 sizes the WebSocket gateway for one board per location plus tablets in the back.

**Reliability.** Availability is measured only during trading hours (06:00 to 23:00). Maintenance happens outside them. NFR-AVL-03 exists because the order board is the business's only order channel during lunch. A board that silently stops updating is worse than one that says it is offline.

**Security and privacy.**
- NFR-SEC-02 keeps the platform in the lightest PCI DSS scope: card data is entered only in the card payment provider's hosted fields or wallet sheets.
- NFR-SEC-05 enforces business isolation in PostgreSQL row-level security, so a missing filter in application code cannot leak another business's orders.
- NFR-PRIV-01 and the allergen filter design ([spec 001](../08-ai-assisted-ba/specs/001-allergen-filter-on-menus/spec.md)) keep health-related preferences off the server.

**Usability, accessibility and localization.**
- NFR-USE-01 keeps the discovery principle that any business is at most two taps from launch.
- NFR-ACC-01 targets WCAG 2.2 AA. The operator is currently a micro-enterprise and exempt from the Austrian accessibility act for services, but it will not stay exempt, and retrofitting is more expensive than building it in (TBD-10).
- NFR-I18N-01 fixes formats that are easy to get wrong, such as "€ 15,70" and "2,4 km", and fixes the "du" and "Sie" register per app.

**Data correctness.**
- NFR-DAT-01 matters on the two daylight-saving change days. On 2026-10-25 the clock goes back at 03:00. A naive local-time slot generator would duplicate the 02:00 to 03:00 slots, and on the spring change it would create slots that do not exist.
- NFR-DAT-02 is the result of [INC-2026-006](../07-operations/incidents/INC-2026-006-stale-allergen-information.md). Menu responses had been cached for 24 hours as if they were static content.

**Maintainability and observability.**
- NFR-MNT-02 lets a VAT rate change, or a business changing its cutoff, happen without a release.
- NFR-OBS-02 lists business anomaly alerts. Technical monitoring alone missed both 2026 incidents, because no service failed.

## 5. NFRs changed by change requests and incidents

| NFR | Change | Origin | SRS version |
|---|---|---|---|
| NFR-DAT-02 | New: allergen, price and availability changes visible within 60 s; no shared cache above 60 s for menu data | CR-005, INC-2026-006 | v1.3 |
| NFR-OBS-02 | Extended: slot over capacity; 3-D Secure p90 above 5 minutes; holds expiring during payment above 5% in 15 minutes; menu cache age above 60 s | CR-005, CR-006, INC-2026-006, INC-2026-011 | v1.3 |
| NFR-AVL-03 | Clarified: polling fallback within 10 s and the German banner text | UAT finding UAT-06 | v1.2 |

## Related documents

- [Software Requirements Specification](SRS.md)
- [Business rules](business-rules.md)
- [Compliance mapping](compliance-mapping.md)
- [System architecture](../03-design/architecture/system-architecture.md)
- [Test strategy and plan](../06-quality/test-strategy-and-plan.md)
- [Incident management process](../07-operations/incident-management-process.md)
