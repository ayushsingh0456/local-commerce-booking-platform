# EP-06 Business Onboarding & Platform Administration: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | LC-DEL-US-06 |
| Version | 1.2 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-06-19 |
| Reviewers | Product Owner, Engineering Lead, QA Lead, Partner Operations Lead, Compliance Lead, Finance Lead |

## Purpose and scope

User stories for EP-06: the public application, verification by Platform Admins, payment onboarding with the card payment provider, the go-live gate, suspension with a statement of reasons, VAT rate maintenance, platform vouchers and the audit trail.

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-06 |
| Name | Business Onboarding & Platform Administration |
| Modules | ONB (Business Onboarding & Platform Administration) |
| Goal | Take a local business from application to Live in five business days or fewer, with verified identity, payment onboarding, a statement of reasons for every suspension, and a complete audit trail. |
| Objectives | OBJ-05 Onboard businesses quickly |
| Business need | BN-06 |
| Functional requirements | FR-ONB-01 to FR-ONB-09 (9) |
| Stories | US-033 to US-038 (6, 26 points) |
| Release | R1 |

## Story list

| Story | Title | Persona | Priority | Points | Sprint | Release |
|---|---|---|---|---|---|---|
| US-033 | Apply to join the platform | PER-03 Elif Aydin (BIZ-ADM) | Must | 5 | S1 | R1 |
| US-034 | Verify a business application | PER-06 Paul Haider (PLT-ADM) | Must | 5 | S2 | R1 |
| US-035 | Complete payment onboarding | PER-03 Elif Aydin (BIZ-ADM) | Must | 5 | S3 | R1 |
| US-036 | Set up the business page and go live | PER-03 Elif Aydin (BIZ-ADM) | Must | 5 | S6 | R1 |
| US-037 | Suspend or reinstate a business with reasons | PER-06 Paul Haider (PLT-ADM) | Must | 3 | S6 | R1 |
| US-038 | Maintain VAT rates and review the audit trail | PER-06 Paul Haider (PLT-ADM) | Must | 3 | S7 | R1 |
| **Total** |  |  |  | **26** |  |  |

Shared test data: applicant Elif Aydin for "Mohnblume Kaffeehaus e.U." (FN 512376k, ATU71234567), applicant Martina Leitner for "Haarstudio Linde Leitner GmbH" (FN 498812t, ATU69876543), Platform Admin Paul Haider.

## Stories

### US-033 · Apply to join Local Commerce & Booking Platform

| Field | Value |
|---|---|
| Epic | EP-06 Business Onboarding & Platform Administration |
| Persona | PER-03 Elif Aydin (BIZ-ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S1 / R1 |
| Requirements | FR-ONB-01 |
| Business rules | BR-042 |
| Dependencies | None |
| Test cases | TC-ONB-001 |

**Story**
As a business owner, I want to apply online with the details I have at hand, so that I can start without a sales call.

**Acceptance criteria**

```gherkin
Scenario: US-033-AC1 Submit a complete application
  Given Elif has verified her email through the link sent at 20:05
  When she submits legal name "Mohnblume Kaffeehaus e.U.", legal form "e.U.", Firmenbuchnummer "FN 512376k", UID "ATU71234567", registered address "Kornblumengasse 9, 1070 Wien", category "Essen & Trinken", types "Café" and "Bäckerei", first location at the same address, contact phone and the accepted partner terms
  Then the application is Applied with UID check "valid"

Scenario Outline: US-033-AC2 Register and UID formats
  When Elif enters <field> "<value>"
  Then the result is "<result>"

  Examples:
    | field             | value        | result                                                            |
    | Firmenbuchnummer  | FN 512376k   | accepted                                                          |
    | Firmenbuchnummer  | 512376k      | Bitte geben Sie die Firmenbuchnummer im Format FN 123456a ein.    |
    | UID-Nummer        | ATU71234567  | accepted                                                          |
    | UID-Nummer        | ATU7123456   | Bitte geben Sie eine gültige UID-Nummer ein (ATU und 8 Ziffern).  |
    | Postleitzahl      | 107          | Bitte geben Sie eine österreichische Postleitzahl (4 Ziffern) ein. |

Scenario: US-033-AC3 Small business without a UID number
  Given a sole trader ticks "Kleinunternehmer ohne UID" and enters a GISA number
  Then the UID field is not required and the business will use tax category Exempt

Scenario: US-033-AC4 Outside the service area
  When an applicant enters a first location in Graz
  Then the application is not submitted and the applicant sees "Die Plattform ist derzeit nur in Wien verfügbar. Wir setzen Sie gerne auf die Warteliste."
```

### US-034 · Verify a business application

| Field | Value |
|---|---|
| Epic | EP-06 Business Onboarding & Platform Administration |
| Persona | PER-06 Paul Haider (PLT-ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-ONB-02 |
| Business rules | BR-042, BR-043 |
| Dependencies | US-033 |
| Test cases | TC-ONB-002 |

**Story**
As a Platform Admin, I want one queue with the evidence for each application, so that I can approve genuine businesses quickly and stop the others.

**Acceptance criteria**

```gherkin
Scenario: US-034-AC1 Queue oldest first with evidence
  When Paul opens the verification queue
  Then applications are listed oldest first with legal name, legal form, register number, UID check result and waiting time in business days

Scenario Outline: US-034-AC2 UID check outcomes
  Given the EU VAT number validation service returns <response> for the UID
  When Paul tries to approve
  Then the result is "<result>"

  Examples:
    | response    | result                                                     |
    | valid       | approved                                                   |
    | invalid     | approve disabled; "UID-Nummer ungültig"                    |
    | unavailable | approve disabled; "Die UID-Prüfung ist noch offen." with a retry |

Scenario: US-034-AC3 Reject or request information with a reason
  When Paul rejects without a reason
  Then he sees "Bitte geben Sie einen Grund an."
  When he requests information with "Bitte laden Sie den Gewerbeschein hoch."
  Then the applicant is emailed and the status is "Info requested"

Scenario: US-034-AC4 Every decision is audited
  When Paul approves Mohnblume Kaffeehaus e.U.
  Then an audit event "verification.decided" records Paul, the decision, the time and the UID check result
```

### US-035 · Complete payment onboarding

| Field | Value |
|---|---|
| Epic | EP-06 Business Onboarding & Platform Administration |
| Persona | PER-03 Elif Aydin (BIZ-ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-ONB-03 |
| Business rules | BR-043 |
| Dependencies | US-034 |
| Test cases | TC-ONB-003 |

**Story**
As an approved café owner, I want to complete the payment provider's checks from the portal, so that I can receive money from app orders.

**Acceptance criteria**

```gherkin
Scenario: US-035-AC1 Start hosted onboarding
  Given Mohnblume Kaffeehaus e.U. is Approved
  When Elif taps "Zahlungen einrichten"
  Then she is redirected to the card payment provider's hosted onboarding with her business pre-filled

Scenario Outline: US-035-AC2 Status from the provider
  When the provider sends "account.verification_updated" with <status>
  Then the portal checklist shows "<label>"

  Examples:
    | status     | label                                        |
    | Pending    | Zahlungen: in Prüfung                        |
    | Verified   | Zahlungen: eingerichtet                      |
    | Restricted | Zahlungen: Angaben fehlen (Link zum Anbieter) |

Scenario: US-035-AC3 Salons and spas skip payment onboarding in R1
  Given Haarstudio Linde Leitner GmbH is Approved
  Then its go-live checklist has no payment step
```

### US-036 · Set up the business page and go live

| Field | Value |
|---|---|
| Epic | EP-06 Business Onboarding & Platform Administration |
| Persona | PER-03 Elif Aydin (BIZ-ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S6 / R1 |
| Requirements | FR-CAT-01, FR-CAT-02, FR-ONB-04 |
| Business rules | BR-007, BR-017, BR-018, BR-023, BR-043 |
| Dependencies | US-035, US-029 |
| Test cases | TC-CAT-005, TC-CAT-010, TC-ONB-004 |

**Story**
As a business owner, I want a checklist that tells me what is missing and a business page that looks like mine, so that I can go live in days, not weeks.

**Acceptance criteria**

```gherkin
Scenario: US-036-AC1 Brand color must be readable
  When Elif sets the brand color "#F4C430"
  Then she sees "Diese Farbe ist auf Weiß schlecht lesbar (Kontrast 1,6:1). Bitte wählen Sie eine dunklere Farbe."
  When she sets "#8A3B12" (contrast 7.7:1)
  Then the color is accepted

Scenario: US-036-AC2 Ordering hours must lie inside opening hours
  Given Café Mohnblume opens 07:30 to 18:00
  When Elif sets ordering hours 07:00 to 17:30
  Then she sees "Bestellzeiten müssen innerhalb der Öffnungszeiten liegen."

Scenario Outline: US-036-AC3 Location settings ranges
  When Elif sets <setting> to <value>
  Then the result is "<result>"

  Examples:
    | setting            | value | result   |
    | slot capacity      | 0     | refused  |
    | slot capacity      | 6     | accepted |
    | slot capacity      | 31    | refused  |
    | lead time          | 4     | refused  |
    | cancellation cutoff | 120  | accepted |

Scenario: US-036-AC4 Publish only when the gate passes
  Given Mohnblume is Approved and payments are Verified, but no item is published
  When Paul tries to publish
  Then he sees "Noch nicht alle Voraussetzungen erfüllt" with "Mindestens ein Artikel mit vollständigen Allergenangaben"
  When Elif publishes "Topfengolatsche" with a confirmed declaration
  Then Paul can publish and the café appears in the app
```

### US-037 · Suspend or reinstate a business with reasons

| Field | Value |
|---|---|
| Epic | EP-06 Business Onboarding & Platform Administration |
| Persona | PER-06 Paul Haider (PLT-ADM) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S6 / R1 |
| Requirements | FR-ONB-05, FR-ONB-08 |
| Business rules | BR-008, BR-044 |
| Dependencies | US-036 |
| Test cases | TC-ONB-005, TC-ONB-008 |

**Story**
As a Platform Admin, I want to suspend a business with a written statement of reasons and reinstate it when resolved, so that customers are protected and the business is treated fairly.

**Acceptance criteria**

```gherkin
Scenario: US-037-AC1 Suspension with a statement of reasons
  Given Nudelbar Sesam is Live with 2 open orders and 1 accepted pre-order for tomorrow
  When Paul suspends it with the statement "Wiederholte Allergenbeschwerden, Prüfung bis 30.09.2026"
  Then Nudelbar Sesam disappears from discovery and search at once
  And the 2 open orders and the pre-order stay on the order board and can be completed
  And the Business Admin receives the statement by email

Scenario: US-037-AC2 A suspension needs a statement
  When Paul suspends without a statement
  Then he sees "Bitte geben Sie eine Begründung an."

Scenario: US-037-AC3 Reinstate
  When Paul reinstates Nudelbar Sesam
  Then it is Live again and listed in discovery

Scenario: US-037-AC4 Private feedback is not public
  Given Sophie sends feedback "3 · Lange gewartet" for a completed order
  Then Paul sees it in the Platform Admin portal
  And it appears nowhere in the customer app or the Business Web Portal
```

### US-038 · Maintain VAT rates and review the audit trail

| Field | Value |
|---|---|
| Epic | EP-06 Business Onboarding & Platform Administration |
| Persona | PER-06 Paul Haider (PLT-ADM) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S7 / R1 |
| Requirements | FR-ONB-06, FR-ONB-07, FR-ONB-09 |
| Business rules | BR-015, BR-035, BR-036, BR-045 |
| Dependencies | US-030 |
| Test cases | TC-ONB-006, TC-ONB-007, TC-ONB-009 |

**Story**
As a Platform Admin, I want to change VAT rates as dated reference data, create platform-funded vouchers and search the audit trail, so that legal changes need no release and every sensitive change can be traced.

**Acceptance criteria**

```gherkin
Scenario: US-038-AC1 A future VAT rate applies by order date
  Given the Reduced category has the rate 10 % valid from 2022-01-01
  When Paul adds a rate of 8 % for Reduced valid from 2027-01-01 in staging
  Then an order placed on 2026-12-31 at 23:59 uses 10 %
  And an order placed on 2027-01-01 at 00:00 uses 8 %

Scenario: US-038-AC2 platform-funded voucher
  Given Paul created "WILLKOMMEN3", € 3,00 off from € 15,00, funded by the platform
  When Sophie uses it on the canonical basket of € 18,70
  Then she pays € 15,70
  And the commission base is € 18,70 and Café Mohnblume is reimbursed € 3,00 in the weekly payout

Scenario: US-038-AC3 Search the audit trail
  When Paul filters the audit trail by business "Konditorei Pomeranze" and action "allergen.confirmed" for 2026-08-14
  Then he sees the change of "Topfentorte (Stück)" from version 3 (A C G) to version 4 (A C G H) at 08:52 with the confirming user

Scenario: US-038-AC4 Audit events cannot be changed
  When anyone attempts to update or delete an audit event through any API
  Then the request is refused and the attempt itself is logged
```

**Notes**
- AC1 uses an invented future rate in staging to test effective dating; it is not a forecast of Austrian law.

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [Process flows](../../03-design/diagrams/process-flows.md)
- [Compliance mapping](../../02-requirements/compliance-mapping.md)
- [Test cases](../../06-quality/test-cases.md)
