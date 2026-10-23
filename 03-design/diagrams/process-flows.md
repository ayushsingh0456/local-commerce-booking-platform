# Process Flows: Local Commerce & Booking Platform

## Document control

| Field | Value |
|---|---|
| Document ID | QT-DES-PF |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Product Owner; Partner Operations Lead; Partner Success Lead; QA Lead |

**Purpose and scope.** BPMN-style swimlane flows for the five processes where the business rules interact most. Lanes are drawn as subgraphs; diamonds are decisions with the rule that decides them. Exception paths are drawn, not footnoted. Runtime message order is in the [sequence diagrams](sequence-diagrams.md); status lifecycles are in the [state machines](state-machines.md).

## PF-1 Food order from cart to handover (TO-BE, R1.1)

Covers EP-03 and the order board side of EP-05. The flow shows the R1.1 re-check at authorization (CR-006).

```mermaid
flowchart TB
  subgraph CUS["Customer (app)"]
    C1["Add items<br/>(allergens shown first)"]
    C2["Choose pickup slot"]
    C3["Checkout: voucher,<br/>payment method"]
    C4["Confirm with bank<br/>(3-D Secure if asked)"]
    C5["Pick alternative slot"]
    C6["Show pickup code<br/>at the counter"]
  end
  subgraph SYS["the platform API"]
    S1["Create hold 10 min<br/>(BR-019)"]
    S2{"Lines, prices and<br/>allergen versions<br/>unchanged? (BR-013)"}
    S3["Authorise payment"]
    S4{"Capacity free at<br/>authorization? (BR-020)"}
    S5["Order Placed<br/>pickup code issued"]
    S6["Offer next 3 slots<br/>for 3 min"]
    S7["Void authorization"]
    S8{"Accepted within<br/>5 min? (BR-022)"}
    S9["Capture payment"]
    S10["Auto-reject and void"]
  end
  subgraph BIZ["Business (POS view)"]
    B1["New order alert"]
    B2{"Accept?"}
    B3["Reject with reason"]
    B4["Preparing, then Ready"]
    B5["Match code, hand over<br/>(Completed)"]
  end
  C1 --> C2 --> S1 --> C3 --> S2
  S2 -->|"No: re-confirm"| C3
  S2 -->|"Yes"| S3 --> C4 --> S4
  S4 -->|"Yes"| S5 --> B1 --> S8
  S4 -->|"No, later slot free"| S6 --> C5
  C5 -->|"Chosen in 3 min"| S5
  C5 -->|"Not chosen"| S7
  S8 -->|"No"| S10
  S8 -->|"Yes"| B2
  B2 -->|"Yes"| S9 --> B4 --> C6 --> B5
  B2 -->|"No"| B3 --> S7
```

Notes:
- The allergen and price check runs twice: when the customer presses the order button (before the bank step) and again inside the authorization transaction. The first check catches almost all changes before the customer confirms anything with the bank.
- Pay-at-pickup orders skip S3, S4 and S9. At B5 staff must also confirm "Bezahlt an der Kassa" (BR-024).

## PF-2 Appointment booking and the day of the appointment

Covers EP-04 and FR-POS-06.

```mermaid
flowchart TB
  subgraph CUS["Customer (app)"]
    A1["Choose service"]
    A2["Choose staff member<br/>or Beliebig"]
    A3["Choose day and start time"]
    A4["Add note, review price,<br/>Zahlungspflichtig buchen"]
    A5["Reminder 24 h before"]
    A6{"Still coming?"}
    A7["Reschedule or cancel<br/>in the app"]
    A8["Call the business"]
  end
  subgraph SYS["the platform API"]
    P1["Compute start times<br/>(BR-026, table 3.5)"]
    P2["Hold staff time 10 min<br/>(BR-028)"]
    P3{"Database constraint<br/>accepts? (BR-028)"}
    P4["Confirmed; assign staff<br/>for Beliebig (BR-027)"]
    P5["Show 3 nearest<br/>free times"]
    P6{"Before the cutoff?<br/>(BR-029)"}
  end
  subgraph BIZ["Business (POS day view)"]
    D1["Check in on arrival"]
    D2["Complete"]
    D3["No-show from<br/>start + 15 min (BR-030)"]
  end
  A1 --> A2 --> P1 --> A3 --> P2 --> A4 --> P3
  P3 -->|"Yes"| P4 --> A5 --> A6
  P3 -->|"No, time taken"| P5 --> A3
  A6 -->|"Yes"| D1 --> D2
  A6 -->|"No"| P6
  P6 -->|"Yes"| A7
  P6 -->|"No"| A8
  A6 -->|"Does not come"| D3
```

## PF-3 Business onboarding to Live

Covers EP-06 and the setup stories in EP-05. The target is 5 business days or fewer (OBJ-05).

```mermaid
flowchart TB
  subgraph APP["Applicant (Business Admin)"]
    O1["Verify email,<br/>submit application"]
    O2["Answer information request"]
    O3["Complete payment onboarding<br/>(food only)"]
    O4["Set up profile, location,<br/>catalog or services"]
    O5["Request go-live"]
  end
  subgraph PLT["Platform Admin"]
    V1["Review in queue:<br/>register, UID, legal form"]
    V2{"Decision"}
    V3["Reject with reason"]
    V4["Check go-live gate<br/>and publish"]
  end
  subgraph EXT["External"]
    E1["VAT number validation"]
    E2["Card payment provider<br/>KYC and bank checks"]
  end
  O1 --> E1 --> V1 --> V2
  V2 -->|"Request info"| O2 --> V1
  V2 -->|"Reject"| V3
  V2 -->|"Approve"| O3
  O3 --> E2
  E2 -->|"Verified (webhook)"| O4
  V2 -->|"Approve, salon or spa"| O4
  O4 --> O5 --> V4
  V4 -->|"Gate fails: show missing items"| O4
```

The gate is table 3.8 in the [business rules](../../02-requirements/business-rules.md). Salons and spas skip payment onboarding in R1 because appointments are paid at the venue.

## PF-4 Allergen change from the kitchen to the customer (after CR-005)

Covers BR-013, FR-CAT-04 and FR-ORD-12. This is the flow that failed in INC-2026-006 at step "Customer sees it".

```mermaid
flowchart TB
  subgraph ADM["Business Admin (portal)"]
    K1["Edit ingredients or allergens<br/>of an item or choice"]
    K2["Confirm declaration<br/>(tick box)"]
  end
  subgraph SYS["the platform API and jobs"]
    K3["New declaration version<br/>and audit event"]
    K4["Menu ETag changes;<br/>shared cache max 60 s"]
    K5{"Open pre-orders<br/>with this item?"}
    K6["Email customers:<br/>free cancellation until preparing"]
    K7{"Cart line version<br/>equals current?"}
  end
  subgraph CUS["Customer (app)"]
    K8["Sees new allergens<br/>within 60 s"]
    K9["At checkout: review<br/>and confirm change"]
    K10["Pays"]
  end
  K1 --> K2 --> K3 --> K4 --> K8
  K3 --> K5
  K5 -->|"Yes"| K6
  K8 --> K7
  K7 -->|"No"| K9 --> K10
  K7 -->|"Yes"| K10
```

## PF-5 Order exceptions: rejection, timeout, cancellation and non-collection

Covers BR-022 to BR-024 and table 3.6a.

```mermaid
flowchart LR
  P["Placed"] -->|"Business rejects<br/>with reason"| R["Rejected:<br/>void"]
  P -->|"No answer in 5 min"| AR["Rejected (auto):<br/>void"]
  P -->|"Customer cancels"| CP["Canceled:<br/>void"]
  P -->|"Business accepts"| A["Accepted:<br/>capture"]
  A -->|"Customer cancels<br/>before cutoff"| CA["Canceled:<br/>full refund"]
  A -->|"After cutoff"| CALL["Customer must call;<br/>business may refund"]
  A --> PR["Preparing"] --> RD["Ready"]
  RD -->|"Code matched"| CO["Completed"]
  RD -->|"Not picked up by<br/>slot end + 60 min"| NC["Not collected:<br/>no automatic refund"]
```

## Related documents

- [Sequence diagrams](sequence-diagrams.md)
- [State machines](state-machines.md)
- [Business rules](../../02-requirements/business-rules.md)
- [Current vs future state](../../01-discovery/current-vs-future-state.md)
- [Software Requirements Specification](../../02-requirements/SRS.md)
