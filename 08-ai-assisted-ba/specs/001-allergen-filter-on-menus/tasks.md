# Tasks: Allergen Filter on Menus

## Document control

| Field | Value |
|---|---|
| Document ID | QT-SPEC-001-TASKS |
| Version | 1.2 |
| Status | v1.1 tasks done (Sprint 5); v1.2 tasks done in S13 for R1.1 |
| Owner | Flutter developer (with the Business Analyst for traceability) |
| Last updated | 2026-09-25 |
| Reviewers | QA Lead, Engineering Lead |

**Input**: Design documents from `specs/001-allergen-filter-on-menus/` ([spec.md](spec.md), [plan.md](plan.md))
**Prerequisites**: plan.md (required), spec.md (required), OpenAPI `Item` schema

### Purpose and scope

The output of `/tasks`: ordered, executable tasks for the AI coding agent. Each task names the files it touches and the SPEC-FR it implements. `[P]` marks tasks that can run in parallel because they touch different files and have no dependency on each other. A developer reviews every agent pull request against the spec before merge.

## Format: `[ID] [P?] Description (SPEC-FR)`

## Phase 3.1: Setup

- [x] T001 Create the feature folders `lib/features/allergen_filter/` and `test/features/allergen_filter/` per plan.md (n/a)
- [x] T002 [P] Add de-AT and en strings for chip, sheet, labels, banner and screen-reader texts to the translation files (SPEC-FR-005, SPEC-FR-010, SPEC-FR-014)
- [x] T003 [P] Add the 14 allergens with letter order and names in `domain/allergen.dart` (SPEC-FR-001, SPEC-FR-014)

## Phase 3.2: Tests first (they must fail before Phase 3.3)

- [x] T004 [P] Table-driven unit tests for `classifyItem` covering rows M1, M2, M3 with the menu-items.csv data: Nussschnecke M1(H), Linsen-Bowl M2(H), Topfengolatsche M3, Limonade M3, Linsen-Bowl with filter H and N M2(H, N) in `classify_item_test.dart` (SPEC-FR-003)
- [x] T005 [P] Unit tests for the configured-line hint: Linsen-Bowl with Cashew-Topping and filter H returns "Contains H" (SPEC-FR-006)
- [x] T006 [P] Edge-case tests: all 14 selected; "contains none" item never labeled; every choice in a required group adds H (SPEC-FR-003)
- [x] T007 [P] Storage tests: persists across restart and sign-out; cleared by "Daten auf diesem Gerät löschen"; migration from no key (SPEC-FR-013)
- [x] T008 [P] Integration test with a recording proxy: browse 3 menus and place an order with filter H; assert no request body, header, query, analytics event or error event contains the selection or a filter flag in `allergen_filter_privacy_test.dart` (SPEC-FR-002, SPEC-FR-016)
- [x] T009 [P] Forbidden-words test over both translation files and all widgets in the feature: "safe", "allergen-free", "suitable", "sicher", "allergenfrei", "geeignet" (SPEC-FR-004)

## Phase 3.3: Core implementation

- [x] T010 Implement `classifyItem` and the configured-line function in `domain/classify_item.dart` until T004 to T006 pass (SPEC-FR-003, SPEC-FR-006)
- [x] T011 Implement `filter_store.dart` on encrypted storage and `filter_provider.dart` deriving classifications per catalog (SPEC-FR-002, SPEC-FR-009, SPEC-FR-013)
- [x] T012 [P] `filter_chip.dart` and `filter_sheet.dart` with "Filter (n)" and one-tap clear (SPEC-FR-001, SPEC-FR-012)
- [x] T013 [P] `menu_row_marking.dart`: dimming plus label plus bold letters; wraps at 200% text; semantics label "contains tree nuts, matches your filter" (SPEC-FR-003, SPEC-FR-010, SPEC-FR-011)
- [x] T014 [P] `choice_marking.dart`: mark choices; "Contains" hint next to Add; adding stays enabled (SPEC-FR-006)
- [x] T015 [P] `filter_banner.dart` with filtered letters, source and traces sentence (SPEC-FR-005)

## Phase 3.4: Integration

- [x] T016 Apply the marking to dish results in search (SPEC-FR-008)
- [x] T017 Telemetry deny-list: the wrapper drops any field derived from the filter provider; lint rule forbids importing the provider in `lib/telemetry/` (SPEC-FR-002, SPEC-FR-016)
- [x] T018 Error-monitoring scrubber rule for the storage key and provider state (SPEC-FR-002)

## Phase 3.5: Freshness (v1.2, after INC-2026-006)

- [x] T019 Revalidate the catalog with If-None-Match on every business page open; re-run classification when the ETag changes (SPEC-FR-009)
- [x] T020 Track `revalidatedAt`; when offline and older than 60 s, show "Menu may not be up to date" in the banner and disable Add (SPEC-FR-015)
- [x] T021 [P] Tests for T019 and T020: changed declaration re-dims within one refresh; offline older than 60 s disables Add (SPEC-FR-009, SPEC-FR-015)

## Phase 3.6: Polish

- [x] T022 [P] Widget and golden tests at 100% and 200% text in de-AT and en (SPEC-FR-011, SPEC-FR-014)
- [x] T023 [P] Performance check: classify a 120-item menu in under 5 ms on the reference Android device (plan.md performance goal)
- [x] T024 Manual accessibility pass with VoiceOver and TalkBack against acceptance scenario 10 (SPEC-FR-010)

## Dependencies

- T001 before everything.
- T004 to T009 before T010 to T015 (tests first).
- T010 before T011, T013, T014 and T016.
- T011 before T017, T018, T019 and T020.
- T019 and T020 before T021.
- Implementation before polish (T022 to T024).

## Parallel example

```text
# After T001, launch together:
Task: "T004 Table-driven unit tests for classifyItem in test/features/allergen_filter/classify_item_test.dart"
Task: "T007 Storage tests in test/features/allergen_filter/filter_store_test.dart"
Task: "T008 Privacy integration test in integration_test/allergen_filter_privacy_test.dart"
Task: "T009 Forbidden-words test in test/features/allergen_filter/forbidden_words_test.dart"
```

## Validation checklist

- [x] Every SPEC-FR has at least one task
- [x] Every test task comes before its implementation task
- [x] Parallel tasks touch different files
- [x] Each task names exact files or a clear location
- [x] Results: TC-DSC-008 and TC-NFR-009 pass in the R1.1 regression

## Related documents

- [Feature specification](spec.md)
- [Implementation plan](plan.md)
- [AI-assisted BA README](../../README.md)
- [Test cases](../../../06-quality/test-cases.md)
