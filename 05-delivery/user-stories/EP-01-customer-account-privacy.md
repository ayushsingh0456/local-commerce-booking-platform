# EP-01 Customer Account & Privacy: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DEL-US-01 |
| Version | 1.2 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-06-19 |
| Reviewers | Product Owner, Engineering Lead, QA Lead, UX Designer, Compliance Lead |

## Purpose and scope

User stories and acceptance criteria for EP-01: guest browsing, sign-up with the minimum data set, sign-in with email, Google or Apple, password reset by code, and the customer's control over profile, consents and deletion. Version 1.2 aligns the messages with SRS v1.2 (German and English for every message).

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-01 |
| Name | Customer Account & Privacy |
| Modules | ACC (Customer Account & Privacy) |
| Goal | Let anyone browse without an account, ask for one only when it is needed, collect the smallest data set a business needs, and give customers full control of consent and deletion. |
| Objectives | OBJ-01 Cut uncollected pickup orders; OBJ-02 Cut appointment no-shows |
| Business need | BN-01 |
| Functional requirements | FR-ACC-01 to FR-ACC-09 (9) |
| Stories | US-001 to US-005 (5, 21 points) |
| Release | R1 |

## Story list

| Story | Title | Persona | Priority | Points | Sprint | Release |
|---|---|---|---|---|---|---|
| US-001 | Browse as a guest and sign in only when needed | PER-01 Sophie Wallner (CUS) | Must | 3 | S1 | R1 |
| US-002 | Create an account with the minimum details | PER-01 Sophie Wallner (CUS) | Must | 5 | S1 | R1 |
| US-003 | Sign in with email, Google or Apple | PER-02 Daniela Costa (CUS) | Must | 5 | S1 | R1 |
| US-004 | Reset a forgotten password with a code | PER-01 Sophie Wallner (CUS) | Must | 3 | S2 | R1 |
| US-005 | Manage my profile, consents and account deletion | PER-02 Daniela Costa (CUS) | Must | 5 | S6 | R1 |
| **Total** |  |  |  | **21** |  |  |

Shared test data: customer Sophie Wallner (`sophie.wallner@example.com`, +43 664 1234567); customer Daniela Costa (`daniela.costa@example.com`, English UI); business user `elif.aydin@mohnblume.example` (Business Admin of Café Mohnblume, a portal account).

## Stories

### US-001 · Browse as a guest and sign in only when needed

| Field | Value |
|---|---|
| Epic | EP-01 Customer Account & Privacy |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S1 / R1 |
| Requirements | FR-ACC-01, FR-DSC-08 |
| Business rules | BR-001 |
| Dependencies | None |
| Test cases | TC-ACC-001, TC-DSC-009 |

**Story**
As a first-time visitor, I want to look at businesses, menus, prices and allergens without creating an account, so that I can decide whether the app is worth signing up for.

**Acceptance criteria**

```gherkin
Scenario: US-001-AC1 A guest sees the whole menu with prices and allergens
  Given Sophie has opened the app without an account
  When she opens Café Mohnblume and taps "Topfengolatsche"
  Then she sees the price "€ 3,40" and "Enthält: A Glutenhaltiges Getreide, C Eier, G Milch oder Laktose"
  And the Add button reads "Melde dich an, um zu bestellen"

Scenario Outline: US-001-AC2 Actions that need an account open the sign-in prompt
  Given Sophie is a guest
  When she <action>
  Then the sign-in prompt opens and nothing is <result>

  Examples:
    | action                                   | result   |
    | taps Add on an item                      | added    |
    | taps the heart on a business card        | saved    |
    | opens the Bookings tab                   | shown    |
    | taps "Termin buchen" on a salon service  | held     |

Scenario: US-001-AC3 Return to the blocked action after signing in
  Given Sophie tapped the heart on Café Mohnblume as a guest and the sign-in prompt opened
  When she signs in successfully
  Then she is back on the Café Mohnblume card
  And the heart action completes without a second tap

Scenario: US-001-AC4 Save and unsave a location
  Given Sophie is signed in
  When she taps the heart on Café Mohnblume and then on Nudelbar Sesam, Neubau
  Then both locations are listed under Profile, Gespeichert
  When she taps the heart on Café Mohnblume again
  Then only Nudelbar Sesam, Neubau remains saved
```

**Notes**
- The guest scope comes from discovery and is unchanged (BR-001). The app keeps no account-bound data for guests.
- The heart is a 44 x 44 pt target with the accessible name "Speichern" or "Gespeichert".

### US-002 · Create an account with the minimum details

| Field | Value |
|---|---|
| Epic | EP-01 Customer Account & Privacy |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S1 / R1 |
| Requirements | FR-ACC-02, FR-ACC-05, FR-ACC-08 |
| Business rules | BR-002, BR-003 |
| Dependencies | None |
| Test cases | TC-ACC-002, TC-ACC-003, TC-ACC-006 |

**Story**
As a new customer, I want to create an account with only the details a business needs to serve me, so that I can order without handing over data I do not want to share.

**Acceptance criteria**

```gherkin
Scenario: US-002-AC1 Create an account with the minimum data set
  Given Sophie opens Sign Up
  When she enters first name "Sophie", last name "Wallner", email "sophie.wallner@example.com", mobile "+43 664 1234567", password "Mohn2026!" twice
  And she ticks "Ich bin 14 Jahre oder älter."
  And she leaves "Marketing-E-Mails" unticked
  And she taps "Registrieren"
  Then the account is created and she is signed in on Home
  And the stored data is exactly first name, last name, email, mobile, password hash, age declaration and locale
  And consents are stored for the terms and privacy versions shown, with marketing email not granted

Scenario Outline: US-002-AC2 Name and password boundaries
  When Sophie enters <field> "<value>" and leaves the field
  Then the result is "<result>"

  Examples:
    | field      | value                                                 | result                                                                   |
    | first name | S                                                     | Bitte gib deinen Vornamen ein (mindestens 2 Buchstaben).                 |
    | first name | Jo                                                    | accepted                                                                 |
    | first name | Anna-Lena O'Brien                                     | accepted                                                                 |
    | first name | Sophie2                                               | Bitte gib deinen Vornamen ein (mindestens 2 Buchstaben).                 |
    | password   | Mohn202                                               | Das Passwort braucht mindestens 8 Zeichen, einen Groß- und einen Kleinbuchstaben und eine Zahl. |
    | password   | Mohn2026                                              | accepted                                                                 |
    | password   | mohn2026                                              | Das Passwort braucht mindestens 8 Zeichen, einen Groß- und einen Kleinbuchstaben und eine Zahl. |
    | password   | Password1                                             | Dieses Passwort ist aus einem Datenleck bekannt. Bitte wähle ein anderes. |

Scenario: US-002-AC3 The age declaration is required
  Given every other field is valid
  When Sophie taps "Registrieren" without ticking "Ich bin 14 Jahre oder älter."
  Then no account is created
  And the tick box shows "Bitte bestätige, dass du mindestens 14 Jahre alt bist."

Scenario Outline: US-002-AC4 Refuse identities that are already in use
  Given <existing>
  When Daniela signs up with <input>
  Then no account is created and she sees "<message>"

  Examples:
    | existing                                                     | input                                   | message                                                            |
    | a customer account uses daniela.costa@example.com            | email daniela.costa@example.com          | An account with this email address already exists. Please sign in. |
    | a customer account uses +43 676 5550123                      | mobile +43 676 5550123                   | This mobile number belongs to another account.           |
    | elif.aydin@mohnblume.example is a Business Web Portal account | email elif.aydin@mohnblume.example      | This email address is registered for a business account. Use a different one for the app.                      |

Scenario: US-002-AC5 Marketing consent is separate from the terms
  Given Sophie signs up and ticks "Marketing-E-Mails"
  Then a consent record "marketing_email, granted, source sign_up" is stored with the current privacy notice version
  And accepting the terms does not depend on the marketing tick box
```

**Notes**
- The field set matches BR-002. Address and date of birth are not collected; discovery showed that a pickup customer never needs an address.
- The age tick box is a self-declaration (DSG section 4); it is not verified, and the Compliance Lead accepted the residual risk.
- The breached-password check uses a k-anonymity range query, so the password never leaves the server in full.

### US-003 · Sign in with email, Google or Apple

| Field | Value |
|---|---|
| Epic | EP-01 Customer Account & Privacy |
| Persona | PER-02 Daniela Costa (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S1 / R1 |
| Requirements | FR-ACC-03 |
| Business rules | BR-004 |
| Dependencies | US-002 |
| Test cases | TC-ACC-004 |

**Story**
As a returning customer, I want to sign in with my email and password or with my Google or Apple account, so that I can get to my bookings quickly on any phone.

**Acceptance criteria**

```gherkin
Scenario: US-003-AC1 Sign in with email and password
  Given Daniela has an account with password "Lisboa2026!"
  When she enters her email and that password and taps "Sign in"
  Then she lands on Home with the greeting "Hi Daniela"

Scenario Outline: US-003-AC2 Lock password sign-in after 5 consecutive failures
  Given Daniela has made <failures> consecutive failed password attempts
  When she enters a wrong password again
  Then the result is "<result>"

  Examples:
    | failures | result                                                                            |
    | 3        | Incorrect email address or password.                                              |
    | 4        | Password sign-in is paused for 15 minutes. Reset your password or continue with Google or Apple.         |

Scenario Outline: US-003-AC3 Lock lasts 15 minutes
  Given Daniela's password sign-in was locked at 08:00:00
  When she signs in with the correct password at <time>
  Then the result is "<result>"

  Examples:
    | time     | result                     |
    | 08:14:59 | still locked               |
    | 08:15:00 | signed in                  |

Scenario: US-003-AC4 Same message for an unknown email and a wrong password
  When someone signs in with "nobody@example.com" and any password
  Then the message is "Incorrect email address or password."
  And the response time does not differ from a wrong password for an existing account by more than 50 ms on average over 100 attempts

Scenario: US-003-AC5 Sign in with Apple on iOS, not offered on Android
  Given Sophie uses an iPhone and Daniela uses an Android phone
  Then Sophie sees "Weiter mit Google" and "Weiter mit Apple"
  And Daniela sees "Continue with Google" only
  When Sophie signs in with Apple using a verified email that matches no account
  Then the app asks for her mobile number and the age declaration before the account is created
```

**Notes**
- Google and Apple sign-in stay available while password sign-in is locked (BR-004).
- Social sign-up still collects mobile number and the age declaration, because the business needs the number (BR-002).

### US-004 · Reset a forgotten password with a code

| Field | Value |
|---|---|
| Epic | EP-01 Customer Account & Privacy |
| Persona | PER-01 Sophie Wallner (CUS) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-ACC-04 |
| Business rules | BR-004 |
| Dependencies | US-003 |
| Test cases | TC-ACC-005 |

**Story**
As a customer who has forgotten my password, I want a code by email to set a new one, so that I can get back into my account without calling support.

**Acceptance criteria**

```gherkin
Scenario: US-004-AC1 Request a code without revealing whether the account exists
  When Sophie requests a code for "sophie.wallner@example.com"
  And someone else requests a code for "unknown@example.com"
  Then both see "Wenn es ein Konto mit dieser E-Mail-Adresse gibt, haben wir dir einen Code geschickt."
  And only Sophie receives an email with a 6-digit code

Scenario Outline: US-004-AC2 The code is valid for 10 minutes
  Given a code was sent to Sophie at 19:00:00
  When she enters it at <time>
  Then the result is "<result>"

  Examples:
    | time     | result                                                  |
    | 19:09:59 | accepted                                                |
    | 19:10:00 | Der Code ist abgelaufen. Bitte fordere einen neuen an.  |

Scenario: US-004-AC3 Five wrong attempts invalidate the code
  Given Sophie has a valid code
  When she enters a wrong code 4 times
  Then she sees "Der Code ist nicht korrekt. Noch 1 Versuch." after the fourth attempt
  When she enters a wrong code a fifth time
  Then she sees "Zu viele Versuche. Bitte fordere einen neuen Code an."
  And the correct code no longer works

Scenario Outline: US-004-AC4 Resend after 60 seconds invalidates the earlier code
  Given the first code was sent at 19:00:00
  When Sophie taps "Code erneut senden" at <time>
  Then the result is "<result>"

  Examples:
    | time     | result                                                                     |
    | 19:00:59 | control disabled showing "Neuer Code in 0:01"                              |
    | 19:01:00 | a new code is sent; the first code is rejected as not correct              |

Scenario: US-004-AC5 Saving the new password ends other sessions
  Given Sophie is signed in on her iPhone and her iPad
  When she completes a reset on the iPhone with the new password "Golatsche7"
  Then the iPad session ends at its next request
  And Sign In shows "Dein Passwort wurde geändert. Bitte melde dich an."
  And reusing the current password as the new password is refused with "Bitte wähle ein Passwort, das du derzeit nicht verwendest."
```

**Notes**
- The non-revealing wording in AC1 changes the discovery wording ("no account found"). The Compliance Lead approved this (SRS 3.2.1).
- Attempt counts use the plural rules of the translation file ("Noch 1 Versuch", "Noch 2 Versuche").

### US-005 · Manage my profile, consents and account deletion

| Field | Value |
|---|---|
| Epic | EP-01 Customer Account & Privacy |
| Persona | PER-02 Daniela Costa (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S6 / R1 |
| Requirements | FR-ACC-06, FR-ACC-07, FR-ACC-08, FR-ACC-09 |
| Business rules | BR-003, BR-005, BR-041 |
| Dependencies | US-002 |
| Test cases | TC-ACC-006, TC-ACC-007, TC-ACC-008, TC-ACC-009 |

**Story**
As a customer, I want to change my details, language and notifications, withdraw consents and delete my account myself, so that I stay in control of my data.

**Acceptance criteria**

```gherkin
Scenario: US-005-AC1 Switch language and push notifications
  Given Daniela uses the app in German
  When she selects "English" in Profile and switches order and appointment push notifications off
  Then the app shows English immediately without a restart
  And her next order confirmation still arrives by email

Scenario: US-005-AC2 Withdraw a consent as easily as it was given
  Given Daniela granted analytics and marketing email
  When she switches both off in Profile, Privacy
  Then new consent records "analytics, not granted" and "marketing_email, not granted" are stored with timestamp and source "profile"
  And the analytics SDK stops sending events in the same session

Scenario Outline: US-005-AC3 Account deletion is refused while something is open
  Given Daniela has <open item>
  When she requests deletion and confirms with her password
  Then the result is "<result>"

  Examples:
    | open item                                                       | result                                                              |
    | an order in status Ready                                        | refused: "You still have open orders or appointments."              |
    | an appointment on 2026-10-20 at 10:00                           | refused: "You still have open orders or appointments."              |
    | only completed orders                                           | accepted: "Your account will be deleted within 30 days."            |

Scenario: US-005-AC4 Deleted accounts keep the records the law requires
  Given Daniela's deletion request was accepted on 2026-09-02
  When the deletion job has run, at the latest on 2026-10-02
  Then her name, email, mobile, consents and device tokens are erased
  And her orders, payments and receipts remain under a pseudonymous customer ID for 7 years
  And signing in with her old email fails with "Incorrect email address or password."

Scenario: US-005-AC5 Data export for an access request
  Given Daniela emails support asking for a copy of her data and support verifies her identity
  When Paul Haider runs the export for her account
  Then a JSON file is produced with profile, consents, orders, appointments and saved locations
  And the export is logged in the audit trail with Paul as actor
```

**Notes**
- Erasure and retention follow BR-005; the data dictionary lists what is pseudonymized.
- Data export is a Platform Admin action in R1 (FR-ACC-09); self-service export is an R2 candidate.

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [Compliance mapping](../../02-requirements/compliance-mapping.md)
- [Test cases](../../06-quality/test-cases.md)
