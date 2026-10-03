# Definition of Ready and Definition of Done: Quartier

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DEL-DOR-DOD |
| Version | 1.3 |
| Status | Active |
| Owner | Business Analyst (with the QA Lead and Engineering Lead) |
| Last updated | 2026-09-25 |
| Reviewers | Product Owner; whole team at the S1 and S8 retrospectives |

**Purpose and scope.** The checklists a story must pass before it enters a sprint (Ready) and before it counts as finished (Done), plus the release-level Done. Items added after an incident or a change request say so.

## 1. Definition of Ready (story)

A story is Ready when the Product Owner, the Business Analyst and the QA Lead agree that:

1. It follows INVEST: it is independent enough to be delivered alone, valuable to a named persona, estimable and small (8 points or fewer), and testable.
2. It names its FRs and business rules, and every rule it uses is worded in [business-rules.md](../02-requirements/business-rules.md).
3. It has 3 to 6 Gherkin scenarios, including boundary rows for every number in the rules it uses (times, amounts, counts).
4. Every user-facing message is written in de-AT and en, using "du" in the customer app and "Sie" in the portals (NFR-I18N-01).
5. Allergen-related stories have been reviewed by the Compliance Lead; VAT, commission and refund stories by the Finance Lead.
6. Dependencies are Done or planned in the same sprint; external sandboxes are available.
7. Wireframe or screen reference attached where the story has UI.
8. Test data exists or is listed in the [test-data README](../06-quality/test-data/README.md).
9. Open questions are either answered or logged as TBD with the R1 behavior.

## 2. Definition of Done (story)

1. Code reviewed and merged; build green.
2. Every acceptance scenario automated or, where automation is not sensible, executed manually with evidence attached.
3. Every business rule touched has at least one automated test that fails when the rule is broken (NFR-MNT-01).
4. API changes are in `openapi.yaml`, the contract tests pass, and the error codes are in the [API guidelines](../04-api/api-guidelines.md).
5. Accessibility checks pass: screen reader labels, 200% text, contrast; status and allergens never by color alone (NFR-ACC-01).
6. No personal data, card data or allergen filter selection in logs, analytics or error events (NFR-OBS-01).
7. The traceability matrix regenerates without errors.
8. Demonstrated at the sprint review against the acceptance criteria.
9. **Added after INC-2026-006 (CR-005):** any data a customer relies on for safety (allergens) or money (prices) has a stated maximum staleness and a test that proves it.
10. **Added after INC-2026-011 (CR-006):** every rule that limits capacity or money has a database enforcement point, a concurrency test, and a business anomaly alert in NFR-OBS-02.

## 3. Definition of Done (release)

1. All stories in scope meet the story Definition of Done.
2. Regression suite green; open defects only Medium or Low, each accepted by the Product Owner.
3. Penetration test with no open Critical or High finding.
4. Both worked examples reproduce exactly (TC-PAY-003, TC-APT-002).
5. Release notes in German for businesses; app store texts in German and English.
6. Runbooks updated; NFR-OBS-02 alerts tested in staging (TC-NFR-013).
7. For releases with database changes: the restore was rehearsed within the last quarter.

## Related documents

- [Epics](epics.md)
- [Release and sprint plan](release-and-sprint-plan.md)
- [Test strategy and plan](../06-quality/test-strategy-and-plan.md)
- [Change request log](change-request-log.md)
