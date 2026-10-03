# Feature Specification: Allergen Filter on Menus

## Document control

| Field | Value |
|---|---|
| Document ID | QT-SPEC-001 |
| Version | 1.2 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Product Owner, Compliance Lead, UX Designer, Flutter developers, QA Lead |

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-04-08 | First draft generated with `/specify` from the input below; reviewed and edited by the Business Analyst |
| 1.1 | 2026-04-15 | Clarification sessions 1 and 2; all `[NEEDS CLARIFICATION]` markers resolved; approved for Sprint 5 |
| 1.2 | 2026-09-25 | Aligned with SRS v1.3 after INC-2026-006: re-evaluation on catalog refresh and an outdated-menu warning (C-11, SPEC-FR-015) |

**Feature Branch**: `001-allergen-filter-on-menus`
**Created**: 2026-04-08
**Status**: Approved; implemented in Sprint 5 (2026-05-04 to 2026-05-15), released with the pilot on 2026-07-01; v1.2 changes ship with R1.1
**Input**: User description: "Allergen filter for the Customer Mobile App. A customer chooses which of the 14 EU allergens to avoid; every menu then marks items that contain them, including items whose option choices can add them. Sources: US-010, FR-DSC-07, FR-DSC-09, BR-010, BR-011, BR-012, NFR-PRIV-01, NFR-ACC-01. Mark anything not covered by these sources as [NEEDS CLARIFICATION]."

### Purpose and scope

This specification defines what the allergen filter must do, for whom and why, in testable terms. It follows the GitHub Spec Kit format:
- this file is the output of `/specify` and the clarification sessions;
- [plan.md](plan.md) is the output of `/plan`;
- [tasks.md](tasks.md) is the output of `/tasks` and is what the AI coding agent implements.

An AI assistant produced the first draft. The Business Analyst owns the content, every mapping to a canonical ID, and every clarification decision (see [AI-assisted BA](../../README.md)).

**In scope:**
- choosing allergens to avoid, on the device;
- marking menu rows, Item Details and dish search results;
- the banner;
- accessibility;
- behavior when menu data refreshes.

**Out of scope:**
- traces and "may contain" statements (TBD-02, TBD-04);
- saving the filter to the account (TBD-07);
- any server-side change. The catalog already carries the allergens of every item and choice (FR-DSC-07).

---

## User Scenarios & Testing *(mandatory)*

### Primary User Story

Daniela Costa (PER-02) has a tree-nut allergy and uses the app in English. She opens the filter chip on a menu, selects "H Tree nuts" and closes the sheet. Every menu she opens now dims items that contain tree nuts and labels them "Contains H". Items that are fine on their own, but have a topping that adds nuts, show "Depending on choice: H". When she opens such an item, the nut topping is marked, so she can choose the other one. Nothing tells her an item is safe. A banner reminds her that the information comes from the business and that traces are not recorded. Her choice never leaves her phone.

### Acceptance Scenarios

Scenarios use the shared synthetic data in [menu-items.csv](../../../06-quality/test-data/menu-items.csv): Café Mohnblume's Topfengolatsche (A C G), Hausgemachte Limonade (contains none), Linsen-Bowl (L; choices Sesam-Topping N, Cashew-Topping H) and Nussschnecke (A C G H).

1. **Given** Daniela has no filter, **When** she taps "Filter" on the Café Mohnblume menu and selects H, **Then** the chip reads "Filter (1)" and the menu re-renders without a network request. (FR-DSC-09)
2. **Given** the filter avoids H, **When** the Café Mohnblume menu shows, **Then** Nussschnecke is dimmed with "Contains H", Linsen-Bowl shows "Depending on choice: H", and Topfengolatsche and Hausgemachte Limonade carry no label. (BR-012, table 3.1b; US-010-AC4)
3. **Given** the filter avoids H, **When** Daniela opens Linsen-Bowl, **Then** "Cashew topping" is marked "H" in bold, and selecting it shows "Contains H" next to the Add button while still allowing her to add it. (FR-DSC-07)
4. **Given** the filter avoids H, **Then** a banner above the menu reads "Filter: without H · Information from the business. Traces are not recorded." (C-06)
5. **Given** the filter avoids H and N, **When** she opens Linsen-Bowl, **Then** both toppings are marked and the row on the menu reads "Depending on choice: H, N". (BR-012)
6. **Given** any filter, **Then** no screen uses the words "safe", "allergen-free", "sicher", "allergenfrei" or "geeignet". (C-03)
7. **Given** the filter avoids H, **When** Daniela searches "schnecke", **Then** the dish result "Nussschnecke · Café Mohnblume" carries "Contains H". (C-07)
8. **Given** the filter avoids H, **When** she browses three menus and places an order, **Then** no API request, analytics event or error report contains the filter selection. (NFR-PRIV-01; US-010-AC5)
9. **Given** the filter avoids H, **When** she reinstalls the app, **Then** the filter is empty. **When** she signs out and back in on the same phone, **Then** the filter is kept. (C-01)
10. **Given** VoiceOver is on, **When** focus reaches Nussschnecke, **Then** it is announced as "Nussschnecke, 3 Euro 20, contains tree nuts, matches your filter". (NFR-ACC-01, C-09)
11. **Given** a guest without an account, **When** they set the filter, **Then** it works exactly as for a signed-in customer. (C-02)
12. **Given** the filter avoids H and the Topfentorte at Konditorei Pomeranze changed from A C G to A C G H, **When** Daniela reopens the business page, **Then** the app revalidates the catalog, the item is dimmed with "Contains H", and the filter re-evaluates without her doing anything. (C-11, NFR-DAT-02)
13. **Given** the phone is offline and the catalog was last revalidated more than 60 seconds ago, **Then** the banner adds "Menu may not be up to date" and the Add button is disabled until the catalog is revalidated. (SPEC-FR-015)

### Edge Cases

| Edge case | Expected behavior | Covered by |
|---|---|---|
| All 14 allergens selected | Every item with any declared allergen is dimmed; items declared "contains none" carry no label | SPEC-FR-003 |
| Item with a required option group where every choice adds H | Row shows "Depending on choice: H"; in Item Details every choice is marked | SPEC-FR-003, SPEC-FR-006 |
| Item added to the cart before the filter was set | Cart line shows "Contains H" in the cart and at checkout | SPEC-FR-003 |
| Business page in another language | Letters stay; names follow the app language | SPEC-FR-014 |
| Salon or spa business | No filter chip (no menu) | SPEC-FR-001 |
| Very small screen at 200% text | Label wraps under the price; never truncated | SPEC-FR-011 |
| App upgrade | Existing selection migrated unchanged | SPEC-FR-013 |

---

## Requirements *(mandatory)*

### Functional Requirements

Each requirement maps to the canonical requirement or rule it elaborates, shown in parentheses. SPEC-FR IDs are local to this feature and never appear in the SRS; the canonical IDs remain the traceability anchor.

- **SPEC-FR-001**: The app MUST let a guest or a customer choose any of the 14 allergens to avoid, from the filter chip on every food menu and from Profile, Allergene. (FR-DSC-09, BR-010)
- **SPEC-FR-002**: The app MUST store the selection only on the device, in encrypted local storage, and MUST NOT send it to the API, analytics, error monitoring or push services. (FR-DSC-09, NFR-PRIV-01, NFR-OBS-01)
- **SPEC-FR-003**: The app MUST classify every item by business rules table 3.1b (M1 contains, M2 depending on choice, M3 no label), using the item's and its available choices' declared allergens from the catalog. (BR-012)
- **SPEC-FR-004**: The app MUST NOT label any item or choice as safe, allergen-free or suitable, in any language. (BR-012, C-03)
- **SPEC-FR-005**: The app MUST show a banner on every filtered menu naming the filtered allergens, the business as the source and that traces are not recorded. (C-06)
- **SPEC-FR-006**: Item Details MUST mark each choice that adds a filtered allergen, and MUST show "Contains" with the letters next to Add when the configured item contains one; adding stays possible. (FR-DSC-07, C-04)
- **SPEC-FR-007**: Dimmed items MUST remain visible and openable; the filter never hides items in R1. (C-05)
- **SPEC-FR-008**: Dish results in search MUST carry the same marking as menu rows. (FR-DSC-02, C-07)
- **SPEC-FR-009**: Classification MUST run on the device without extra API calls, and MUST run again whenever catalog data is refreshed. (NFR-PERF-02, C-11)
- **SPEC-FR-010**: Screen readers MUST announce the marking in words, for example "contains tree nuts, matches your filter". (NFR-ACC-01, C-09)
- **SPEC-FR-011**: The marking MUST NOT rely on color: dimming is combined with a text label and bold letters, and labels wrap at 200% text size. (NFR-ACC-01)
- **SPEC-FR-012**: The filter MUST be clearable in one tap; the chip shows the number of selected allergens. (FR-DSC-09)
- **SPEC-FR-013**: The selection MUST persist across restarts, sign-out and app upgrades, and be removed on uninstall or by "Daten auf diesem Gerät löschen" in Profile. (C-01)
- **SPEC-FR-014**: Allergen names MUST follow the app language; letters are identical in German and English. (NFR-I18N-01, C-10)
- **SPEC-FR-015**: When the displayed catalog has not been revalidated in the last 60 seconds because the phone is offline, the banner MUST add "Menu may not be up to date" and Add MUST be disabled until revalidation succeeds. (NFR-DAT-02, C-11)
- **SPEC-FR-016**: No analytics event, experiment or crash breadcrumb MAY reference whether a filter is active or which allergens it contains. (NFR-PRIV-01, C-08)

### Key Entities

| Entity | Where | What it represents | Key attributes |
|---|---|---|---|
| Filter selection | Device only | The allergens a person wants to avoid | Set of letters from BR-010; updated-at timestamp |
| Catalog item | API response (`getCatalog`) | A menu item as published | `itemId`, `allergens.codes`, `allergens.declarationVersion`, `choiceAddsAllergens`, option groups |
| Option choice | API response | A choice inside an option group | `choiceId`, `allergens.codes` |
| Item classification | Device, computed | Result of table 3.1b for one item and one filter | M1, M2 or M3; matched letters |
| Catalog freshness | Device | When the catalog was last revalidated | ETag, revalidated-at timestamp |

---

## Clarifications

The first `/specify` draft contained 12 `[NEEDS CLARIFICATION]` markers. The Business Analyst removed 2 that the sources answered outright (listed at the end of this section) and kept 10 as C-01 to C-10. C-11 was added in version 1.2. Each was resolved with the role that owns the decision.

### Session 2026-04-09 (Product Owner, Compliance Lead, UX Designer, Business Analyst)

| ID | Marker in the draft | Decision | Decided by | Affects |
|---|---|---|---|---|
| C-01 | [NEEDS CLARIFICATION: where is the filter stored, and does it sync across devices?] | On the device only. Syncing to the account would process health data on the server and need explicit consent under GDPR Article 9(2)(a); deferred (TBD-07, DEC-10). | Compliance Lead | SPEC-FR-002, SPEC-FR-013 |
| C-02 | [NEEDS CLARIFICATION: can guests use the filter?] | Yes. Device-only storage needs no account, and guests are exactly the people deciding whether the app is safe for them. | Product Owner | Scenario 11 |
| C-03 | [NEEDS CLARIFICATION: how do we label items without the filtered allergens? The draft proposed "allergenfrei".] | No label at all. Quartier cannot vouch for a recipe or for cross-contact, so the absence of a declared allergen is shown as the absence of a warning, never as a promise. | Compliance Lead | SPEC-FR-004 |
| C-04 | [NEEDS CLARIFICATION: should choices that add a filtered allergen be blocked?] | Marked, not blocked. Customers order for others too, and blocking would hide information. | Product Owner, Compliance Lead | SPEC-FR-006 |
| C-05 | [NEEDS CLARIFICATION: hide or dim matching items?] | Dim and label. Hiding changes the menu structure and makes people think an item does not exist. A "hide matching" toggle is an R2 idea. | UX Designer | SPEC-FR-007 |
| C-06 | [NEEDS CLARIFICATION: how do we handle "may contain traces"?] | Not structured in R1 (TBD-02, TBD-04). The banner says so explicitly. | Compliance Lead | SPEC-FR-005 |

### Session 2026-04-15 (Product Owner, Compliance Lead, Flutter developers, QA Lead, Business Analyst)

| ID | Marker in the draft | Decision | Decided by | Affects |
|---|---|---|---|---|
| C-07 | [NEEDS CLARIFICATION: does the filter apply to search?] | Yes, to dish results only; business results are not marked. | Product Owner | SPEC-FR-008 |
| C-08 | [NEEDS CLARIFICATION: can we measure filter usage?] | No. Even a yes/no flag reveals that a person has an allergy. Adoption is estimated from the customer panel instead. | Compliance Lead | SPEC-FR-016 |
| C-09 | [NEEDS CLARIFICATION: how is the marking announced to screen readers?] | Full words: "contains tree nuts, matches your filter"; letters are not read alone. | UX Designer | SPEC-FR-010 |
| C-10 | [NEEDS CLARIFICATION: are allergen letters translated?] | No. Letters are the Austrian codes in both languages; names are translated. | Business Analyst | SPEC-FR-014 |

### Update 2026-09-25 (SRS v1.3 alignment after INC-2026-006)

| ID | Question | Decision | Decided by | Affects |
|---|---|---|---|---|
| C-11 | What does the filter evaluate when menu data changes or may be outdated? | It always evaluates the catalog on screen. The app revalidates the catalog on every business page open and re-runs the classification on refresh. If the phone is offline and the data is older than 60 seconds, the banner warns and ordering waits. Checkout re-confirmation (US-020) covers changes after an item was added. | Business Analyst, Compliance Lead | SPEC-FR-009, SPEC-FR-015 |

Removed markers (answered by sources): which allergens exist and their letters (BR-010), and whether "contains none" counts as declared (BR-011).

---

## Review & Acceptance Checklist

### Content Quality

- [x] No implementation details in requirements (storage technology and widgets are in plan.md)
- [x] Focused on customer value and on the compliance constraints
- [x] Reviewed by the customer panel's allergy participants in UAT-01
- [x] All mandatory sections completed

### Requirement Completeness

- [x] No `[NEEDS CLARIFICATION]` markers remain open (C-01 to C-11 resolved)
- [x] Requirements are testable; each maps to a canonical FR, BR or NFR
- [x] Success is measurable: no network transmission of the filter (TC-DSC-008), 60-second freshness (TC-NFR-012)
- [x] Scope is bounded: traces, account sync and hiding are excluded
- [x] Dependencies identified: catalog fields from FR-DSC-07; US-020 for checkout

### Business Analyst sign-off checks

- [x] Every SPEC-FR traces to an existing canonical ID; no new FR, BR or US IDs created
- [x] Acceptance scenarios 2 and 8 match US-010-AC4 and US-010-AC5 in substance
- [x] Compliance Lead confirmed C-01, C-03, C-06 and C-08
- [x] AI-drafted content reviewed line by line; corrections recorded in the [AI-assisted BA README](../../README.md)

## Execution Status

- [x] User description parsed
- [x] Key concepts extracted
- [x] Ambiguities marked
- [x] User scenarios defined
- [x] Requirements generated
- [x] Entities identified
- [x] Review checklist passed

## Related documents

- [Implementation plan](plan.md)
- [Tasks](tasks.md)
- [AI-assisted BA README](../../README.md)
- [EP-02 stories (US-010)](../../../05-delivery/user-stories/EP-02-discovery-menus-allergens.md)
- [Business rules table 3.1](../../../02-requirements/business-rules.md)
- [INC-2026-006 Stale allergen information](../../../07-operations/incidents/INC-2026-006-stale-allergen-information.md)
- [Compliance mapping](../../../02-requirements/compliance-mapping.md)
