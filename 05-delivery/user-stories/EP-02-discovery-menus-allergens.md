# EP-02 Discovery, Menus & Allergens: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DEL-US-02 |
| Version | 1.3 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Product Owner, Engineering Lead, QA Lead, UX Designer, Compliance Lead |

## Purpose and scope

User stories for EP-02: Home, search, category listing, the business page with its menu, and allergen information including the allergen filter. Version 1.3 adds US-010-AC6 for the allergen display after a declaration change (CR-005, SRS v1.3).

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-02 |
| Name | Discovery, Menus & Allergens |
| Modules | DSC (Discovery, Menus & Allergens) |
| Goal | Get a customer from app launch to the right business page within two taps, show every business with its own look and with neutral ranking, and show complete allergen information before anything can be added. |
| Objectives | OBJ-03 Take bookings when the phone cannot; OBJ-04 Complete and current allergen information |
| Business need | BN-02 |
| Functional requirements | FR-DSC-01 to FR-DSC-09 (9) |
| Stories | US-006 to US-010 (5, 24 points) |
| Release | R1 |

## Story list

| Story | Title | Persona | Priority | Points | Sprint | Release |
|---|---|---|---|---|---|---|
| US-006 | Find businesses near me from Home | PER-01 Sophie Wallner (CUS) | Must | 5 | S2 | R1 |
| US-007 | Search for a business, dish or service | PER-02 Daniela Costa (CUS) | Must | 3 | S3 | R1 |
| US-008 | Browse a category by business type | PER-01 Sophie Wallner (CUS) | Must | 3 | S2 | R1 |
| US-009 | Explore a business and what it sells | PER-01 Sophie Wallner (CUS) | Must | 5 | S2 | R1 |
| US-010 | Check allergens before adding an item | PER-02 Daniela Costa (CUS) | Must | 8 | S3 | R1 |
| **Total** |  |  |  | **24** |  |  |

Shared test data: Sophie's location is Neubaugasse, 1070 Wien. Café Mohnblume (L-1001) is 0.4 km away, Nudelbar Sesam Neubau (L-1004) 0.7 km, Konditorei Pomeranze (L-1002) 1.1 km, Haarstudio Linde (L-2001) 0.6 km, Massageatelier Ruhepol (L-3001) 2.4 km. Menu data are in [menu-items.csv](../../06-quality/test-data/menu-items.csv).

## Stories

### US-006 · Find businesses near me from Home

| Field | Value |
|---|---|
| Epic | EP-02 Discovery, Menus & Allergens |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-DSC-01, FR-DSC-04 |
| Business rules | BR-006, BR-007, BR-008 |
| Dependencies | None |
| Test cases | TC-DSC-001, TC-DSC-002 |

**Story**
As a customer in a hurry at lunchtime, I want Home to show me what is near me and what I already have going, so that I reach the right business page in two taps.

**Acceptance criteria**

```gherkin
Scenario: US-006-AC1 Home sections in order for a signed-in customer
  Given Sophie is signed in, has location permission and has an order in status Preparing at Café Mohnblume
  When she opens the app
  Then Home shows, in order: "Hallo Sophie" with the location chip "Neubau", the search bar, the cards "Essen & Trinken", "Haare & Beauty" and "Wellness & Massage", an Ongoing card "Café Mohnblume · In Zubereitung · Abholung 12:15", and Near you
  And Near you lists Café Mohnblume (0,4 km), Haarstudio Linde (0,6 km), Nudelbar Sesam Neubau (0,7 km), Konditorei Pomeranze (1,1 km), Massageatelier Ruhepol (2,4 km) in that order

Scenario: US-006-AC2 Without location permission, sorting falls back to name and the header states it
  Given Sophie has denied location permission
  When she opens Home
  Then Near you is swapped for a card "Standort aktivieren, um Geschäfte in deiner Nähe zu sehen"
  And "Alle anzeigen" opens a list headed "5 Geschäfte · sortiert nach Name" starting with "Café Mohnblume"

Scenario Outline: US-006-AC3 Open state is always in words
  Given Café Mohnblume opens 07:30 to 18:00 on Thursday 2026-09-17
  When Sophie looks at the card at <time>
  Then the card shows "<label>"

  Examples:
    | time  | label             |
    | 07:29 | Öffnet um 07:30   |
    | 07:30 | Geöffnet bis 18:00 |
    | 17:59 | Geöffnet bis 18:00 |
    | 18:00 | Heute geschlossen |

Scenario: US-006-AC4 Cards carry no ratings, scores or sponsored markers
  When Sophie looks at any business card in Home, a listing or search
  Then the card shows logo or initials, name, type, distance, open state and the heart only
  And no element on the card shows stars, a score, a review count or "Empfohlen"
```

**Notes**
- A dated closure (for example 2026-12-24 "Heute geschlossen") overrides weekly hours (BR-007).
- No "coming soon" labels on categories; all three categories are live in R1.

### US-007 · Search for a business, dish or service

| Field | Value |
|---|---|
| Epic | EP-02 Discovery, Menus & Allergens |
| Persona | PER-02 Daniela Costa (CUS) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-DSC-02 |
| Business rules | BR-009 |
| Dependencies | US-006 |
| Test cases | TC-DSC-007 |

**Story**
As a customer who knows what I want, I want to search for it directly, so that I do not have to browse.

**Acceptance criteria**

```gherkin
Scenario Outline: US-007-AC1 Search starts at 2 characters
  When Daniela types "<text>" into search
  Then <result>

  Examples:
    | text | result                                       |
    | k    | no search is run and recent searches stay    |
    | ka   | results appear grouped under Businesses, Dishes and Services |

Scenario: US-007-AC2 Accent-insensitive matching across groups
  When Daniela types "kaese"
  Then Dishes includes "Schinken-Käse-Sandwich · Café Mohnblume"
  And a typed "Käse" returns the same results

Scenario: US-007-AC3 Tapping a dish jumps to it on the business page
  When Daniela taps "Schinken-Käse-Sandwich · Café Mohnblume"
  Then the Café Mohnblume page opens scrolled to the item, with its category chip selected

Scenario: US-007-AC4 Empty and failed searches
  When Daniela searches for "sushi"
  Then she sees 'No matches for "sushi". Check the spelling or try a broader word.'
  When the search service fails
  Then she sees "Search is not responding. Try again in a moment." with a Retry control
```

**Notes**
- Hidden items and businesses that are not Live are never returned (BR-006, BR-014).
- At most 20 results per group; the group header shows "20+" when more exist.

### US-008 · Browse a category by business type

| Field | Value |
|---|---|
| Epic | EP-02 Discovery, Menus & Allergens |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-DSC-03 |
| Business rules | BR-006, BR-007 |
| Dependencies | US-006 |
| Test cases | TC-DSC-006 |

**Story**
As a customer who wants "a bakery near me", I want to filter a category by business type and open status, so that I see only places I can use now.

**Acceptance criteria**

```gherkin
Scenario: US-008-AC1 Type chips filter immediately
  Given Sophie opens "Essen & Trinken"
  Then the chips "Alle", "Restaurant", "Café", "Bäckerei" are shown with "Alle" selected
  When she taps "Bäckerei"
  Then only Konditorei Pomeranze is listed and the line reads "1 Geschäft · sortiert nach Entfernung"

Scenario: US-008-AC2 Open now filter
  Given it is 06:45 and only Konditorei Pomeranze has opened (from 06:30)
  When Sophie switches on "Jetzt geöffnet" in "Essen & Trinken"
  Then only Konditorei Pomeranze is listed

Scenario: US-008-AC3 A business with two locations appears once per location
  When Sophie opens "Essen & Trinken" with "Alle"
  Then "Nudelbar Sesam · Neubau" and "Nudelbar Sesam · Mariahilf" are separate cards with their own distance

Scenario: US-008-AC4 Empty result
  Given Sophie selected "Restaurant" and "Jetzt geöffnet" at 06:45
  Then she sees "Hier gibt es noch nichts. Probiere eine andere Art oder entferne Filter."
```

### US-009 · Explore a business and what it sells

| Field | Value |
|---|---|
| Epic | EP-02 Discovery, Menus & Allergens |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-DSC-05, FR-DSC-06 |
| Business rules | BR-006, BR-010, BR-014, BR-015 |
| Dependencies | US-008 |
| Test cases | TC-DSC-003, TC-DSC-005 |

**Story**
As a customer, I want each business to look like itself and to show everything it sells with prices including VAT, so that I can choose with confidence and know who I am buying from.

**Acceptance criteria**

```gherkin
Scenario: US-009-AC1 Business page sections
  When Sophie opens Café Mohnblume
  Then she sees the cover, the logo, "Café · Geöffnet bis 18:00 · 0,4 km", the address "Kornblumengasse 9, 1070 Wien"
  And About shows the legal name "Mohnblume Kaffeehaus e.U.", the register number "FN 512376k", the UID number "ATU71234567", phone and email
  And the chips "Beliebt", "Frühstück", "Mehlspeisen", "Getränke" stick to the top while scrolling
  And headings and buttons use the business's brand color

Scenario: US-009-AC2 Menu rows show VAT-inclusive prices and allergen letters
  Then the row "Topfengolatsche" shows "€ 3,40" and "A C G"
  And the row "Hausgemachte Limonade" shows "€ 3,90" and no allergen letters
  And the row "Linsen-Bowl" shows "€ 9,80" and "L + je nach Auswahl"

Scenario: US-009-AC3 Sold-out and hidden items
  Given Tobias marked "Mohnkuchen" sold out and "Mitarbeiter-Frühstück" is hidden from the app
  Then "Mohnkuchen" is dimmed with "Ausverkauft" and has no add action
  And "Mitarbeiter-Frühstück" does not appear on the page or in search

Scenario: US-009-AC4 Location selector for a business with two locations
  When Sophie opens Nudelbar Sesam from the Neubau card
  Then the selector shows "Neubau" and a list with "Mariahilf"
  When she selects "Mariahilf"
  Then address, hours and menu switch to the Mariahilf location

Scenario: US-009-AC5 A closed business keeps its menu readable
  Given it is 19:30 and Café Mohnblume is closed
  Then the whole menu is readable and items can be opened
  And the restriction applies at checkout, not on the page
```

**Notes**
- The brand color is accepted in the portal only with a contrast of at least 4.5:1 against white (FR-CAT-01), so it can be used for text.

### US-010 · Check allergens before adding an item

| Field | Value |
|---|---|
| Epic | EP-02 Discovery, Menus & Allergens |
| Persona | PER-02 Daniela Costa (CUS) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-DSC-07, FR-DSC-09 |
| Business rules | BR-010, BR-011, BR-012 |
| Dependencies | US-009, US-029 |
| Test cases | TC-DSC-004, TC-DSC-008 |

**Story**
As a customer with a tree-nut allergy, I want to see an item's allergens, and those added by each choice, before I can add it, and I want to mark items I must avoid, so that I never order something I cannot eat by accident.

**Acceptance criteria**

```gherkin
Scenario: US-010-AC1 Allergens shown above the options and never collapsed
  When Daniela opens "Linsen-Bowl" at Café Mohnblume
  Then the popup shows "Contains: L Celery. Depending on your choice also: H Tree nuts, N Sesame." above the option group "Topping"
  And the choice "Cashew topping + € 0.90" shows "H" and "Sesame topping + € 0.50" shows "N"

Scenario Outline: US-010-AC2 Declaration states
  When Daniela opens "<item>"
  Then the allergen line reads "<text>"

  Examples:
    | item                    | text                                                          |
    | Topfengolatsche         | Contains: A Cereals containing gluten, C Eggs, G Milk or lactose |
    | Hausgemachte Limonade   | Contains none of the 14 main allergens.                      |
    | Nussschnecke            | Contains: A Cereals containing gluten, C Eggs, G Milk or lactose, H Tree nuts |

Scenario: US-010-AC3 Reference sheet of all 14 allergens
  When Daniela taps the information control next to the allergen line
  Then a sheet lists A, B, C, D, E, F, G, H, L, M, N, O, P, R with their names
  And it ends with "If you have a severe allergy, also speak to the business about traces."

Scenario Outline: US-010-AC4 Allergen filter marks items and never says safe
  Given Daniela has set the filter to avoid "H Tree nuts"
  When she opens the Café Mohnblume menu
  Then the row "<item>" shows "<marking>"

  Examples:
    | item                  | marking                                    |
    | Nussschnecke          | dimmed, "Contains H"                       |
    | Linsen-Bowl           | not dimmed, "Depending on choice: H"       |
    | Topfengolatsche       | no label                                   |
    | Hausgemachte Limonade | no label                                   |

Scenario: US-010-AC5 The filter stays on the phone
  Given Daniela has set the filter to avoid H
  When she browses three menus and places an order
  Then no API request, analytics event or error report contains her filter selection
  And after she reinstalls the app the filter is empty

Scenario: US-010-AC6 Allergens that changed are shown as current within 60 seconds
  Given Daniela is viewing Konditorei Pomeranze, where "Topfentorte (Stück)" shows "A C G"
  When the Business Admin of Konditorei Pomeranze confirms version 4 with "A C G H" at 08:52:00
  Then a refreshed menu at 08:53:00 shows "A C G H"
```

**Notes**
- AC4 and AC5 implement the allergen filter (FR-DSC-09, Should) specified in [spec 001](../../08-ai-assisted-ba/specs/001-allergen-filter-on-menus/spec.md). They were carried over from S3 to S5 when the filter clarifications took a second session; AC1 to AC3 shipped in S3.
- AC6 was added by CR-005 after [INC-2026-006](../../07-operations/incidents/INC-2026-006-stale-allergen-information.md); the checkout side of the change is US-020.
- Out of scope: "may contain traces" statements (TBD-02, TBD-04).

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [Wireframe m-01 Menu and item details](../../03-design/wireframes/m-01-menu-and-item-details.svg)
- [Allergen filter spec](../../08-ai-assisted-ba/specs/001-allergen-filter-on-menus/spec.md)
- [Test cases](../../06-quality/test-cases.md)
