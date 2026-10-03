# Implementation Plan: Allergen Filter on Menus

## Document control

| Field | Value |
|---|---|
| Document ID | QT-SPEC-001-PLAN |
| Version | 1.2 |
| Status | Approved |
| Owner | Flutter developer (technical content), Business Analyst (traceability and constitution check) |
| Last updated | 2026-09-25 |
| Reviewers | Engineering Lead, QA Lead, Compliance Lead |

**Branch**: `001-allergen-filter-on-menus` | **Date**: 2026-04-16 (v1.1), 2026-09-25 (v1.2) | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/001-allergen-filter-on-menus/spec.md`

### Purpose and scope

The output of `/plan`: how the allergen filter is built in the Customer Mobile App, checked against the project constitution, and broken into phases for [tasks.md](tasks.md). It is written for the AI coding agent and the reviewing developer.

## Summary

The filter is a client-only feature. A pure classification function implements business rules table 3.1b over catalog data the app already receives. A small encrypted store keeps the selection. UI components render labels, dimming and the banner. Guards keep the selection out of every outbound channel. No API or database change is needed; the `choiceAddsAllergens` field and the choice-level allergen codes already exist in `getCatalog` (FR-DSC-07).

## Technical Context

| Item | Value |
|---|---|
| Language and framework | Dart, Flutter (stable channel at the time of S5) |
| State management | The app's existing state-management pattern; one provider for the filter selection, derived per menu |
| Storage | Encrypted key-value storage backed by the iOS Keychain or Android Keystore; key `allergen_filter.v1` |
| Data source | `GET /v1/locations/{locationId}/catalog` with ETag (v1.2: revalidated on every business page open) |
| Testing | Dart unit tests (table-driven), widget tests, integration test with a recording proxy, accessibility checks (semantics tree) |
| Target platforms | iOS 16 or later, Android 9 or later (NFR-CMP-01) |
| Performance goal | Classification of a 120-item menu in under 5 ms on the reference Android device; no frame drop when toggling |
| Constraints | No network calls for the filter; no analytics; strings in de-AT and en |

## Constitution Check

The project constitution is a short list of non-negotiable principles kept in the repository for AI agents. It was checked before Phase 0 and again after Phase 1.

| Principle | How this plan complies | Status |
|---|---|---|
| I. Privacy by design: health-related data never leaves the device without explicit consent | Selection stored locally only; analytics and error-reporting guards (tasks T014, T015) | Pass |
| II. Never claim safety | No "safe" or "allergen-free" strings; a test fails on forbidden words in both languages (T009) | Pass |
| III. Rules are implemented from the canonical decision tables | `classifyItem` is generated test-first from table 3.1b rows M1 to M3 (T004 to T006) | Pass |
| IV. Accessible by default (WCAG 2.2 AA) | Semantics labels; no color-only signals; 200% text tests (T011, T012) | Pass |
| V. Every behavior traces to a canonical ID | Each task names its SPEC-FR, and each SPEC-FR names its FR, BR or NFR | Pass |

## Project Structure

```text
lib/features/allergen_filter/
  domain/allergen.dart              # 14 codes, order and names (BR-010)
  domain/classify_item.dart         # table 3.1b, pure function
  data/filter_store.dart            # encrypted local storage
  state/filter_provider.dart        # selection and derived classifications
  ui/filter_chip.dart               # "Filter (n)"
  ui/filter_sheet.dart              # choose allergens
  ui/menu_row_marking.dart          # dimming, label, bold letters
  ui/filter_banner.dart             # source, traces, outdated-menu warning
  ui/choice_marking.dart            # Item Details choices and Add hint
test/features/allergen_filter/
  classify_item_test.dart
  forbidden_words_test.dart
  filter_store_test.dart
  widgets/menu_row_marking_test.dart
integration_test/allergen_filter_privacy_test.dart
```

## Phase 0: Research

| Question | Finding |
|---|---|
| Does the catalog already carry choice-level allergens? | Yes: `optionGroups[].choices[].allergens.codes` and `choiceAddsAllergens` (OpenAPI `Item`) |
| Can analytics and error monitoring be guarded centrally? | Yes: both go through one telemetry wrapper; a deny-list of state keys can be enforced there and in a lint rule |
| Encrypted storage on both platforms? | Yes: Keychain and Keystore through a maintained plugin already used for tokens |

## Phase 1: Design

**Classification function (table 3.1b).**

```text
classifyItem(itemCodes, choiceCodesList, filter):
  matchedOwn = itemCodes intersect filter
  if matchedOwn not empty: return M1(matchedOwn)
  matchedChoices = union(choiceCodesList) intersect filter
  if matchedChoices not empty: return M2(matchedChoices)
  return M3
```

**Configured line** (Item Details and cart): `itemCodes union selectedChoiceCodes`, intersected with the filter, gives the "Contains" hint next to Add (SPEC-FR-006).

**Freshness (v1.2).** The provider recomputes classifications whenever the catalog's ETag changes. A `revalidatedAt` timestamp older than 60 seconds while offline sets `menuMayBeOutdated`, which the banner and the Add button read (SPEC-FR-015).

**Contracts.** No API change. A contract test asserts that `getCatalog` returns `allergens.codes` for every item and choice and that `containsNone` is true only with empty codes (BR-011).

## Phase 2: Task Planning Approach

- Tests first: decision-table unit tests, forbidden-word test, privacy integration test.
- Then domain, storage and state, then UI, then guards, then polish.
- Tasks that touch different files are marked `[P]` for parallel execution by the agent.
- Every task names its SPEC-FR so that the reviewer can trace the diff.

## Complexity Tracking

| Deviation | Why needed | Simpler alternative rejected because |
|---|---|---|
| A lint rule in addition to the telemetry deny-list | Prevents a future developer from logging the selection outside the wrapper | Code review alone missed a similar case in the cart module during S4 |

## Progress Tracking

- [x] Phase 0: Research complete
- [x] Phase 1: Design complete
- [x] Phase 2: Task planning complete (in tasks.md)
- [x] Constitution check passed (initial and post-design)
- [x] All clarifications resolved
- [x] v1.2 freshness changes planned (T019 to T021)

## Related documents

- [Feature specification](spec.md)
- [Tasks](tasks.md)
- [AI-assisted BA README](../../README.md)
- [OpenAPI specification](../../../04-api/openapi.yaml)
- [System architecture](../../../03-design/architecture/system-architecture.md)
