# Real Estate Lead Management — System Architecture

This document presents one end-to-end architecture for deploying the AI Lead Management System with a real estate agency. It maps the existing automation pattern to property-buyer and renter inquiries. Property-specific form fields and client integrations are configured during each agency's onboarding.

## Complete Lead Journey

```mermaid
flowchart TB
    A(["PROPERTY BUYER / RENTER<br/>Submits a property inquiry"])
    B["LEAD INTAKE<br/>Agency form or connected lead source<br/>Name • Contact • Property requirements"]
    C["n8n WEBHOOK<br/>Receive inquiry"]
    D{"DATA VALIDATION<br/>Are required details present?"}
    X(["INVALID INQUIRY<br/>Stop processing"])

    E["PROPERTY REQUIREMENTS<br/>Normalize property type • Area • Budget<br/>Buy / Rent • Preferred timeline"]
    F["GROQ AI QUALIFICATION<br/>Understand intent and buying readiness"]
    G["STRUCTURED AI RESULT<br/>HOT / WARM / COLD<br/>Reason + inquiry summary"]
    H["MERGE LEAD RECORD<br/>Customer details + property needs + AI result"]
    I[("GOOGLE SHEETS<br/>Create lead record<br/>Unique Lead ID • Initial status: Pending")]
    J{"LEAD ROUTING<br/>AI classification"}

    K["COLD LEAD<br/>Update status: Not Required"]
    K1(["COLD COMPLETE<br/>No approval • No email"])

    L["HOT / WARM LEAD<br/>Send to human review"]
    M{"AGENT / MANAGER<br/>Approval decision"}

    N["REJECTED<br/>Update matching Lead ID<br/>Status: Rejected"]
    N1(["CLOSED<br/>No email sent"])

    O["APPROVED<br/>Update matching Lead ID<br/>Status: Approved"]
    P["GROQ AI FOLLOW-UP<br/>Draft a personalized property inquiry reply"]
    Q["GMAIL<br/>Send approved response"]
    R(["FOLLOW-UP SENT<br/>Lead record remains tracked"])

    A --> B --> C --> D
    D -->|No| X
    D -->|Yes| E --> F --> G --> H --> I --> J
    J -->|COLD| K --> K1
    J -->|HOT or WARM| L --> M
    M -->|Reject| N --> N1
    M -->|Approve| O --> P --> Q --> R

    classDef source fill:#eaf2ff,stroke:#4778b8,color:#17365d,stroke-width:1.5px
    classDef process fill:#f4f6f8,stroke:#7b8794,color:#202b36,stroke-width:1.3px
    classDef ai fill:#f1eaff,stroke:#8560b3,color:#392052,stroke-width:1.5px
    classDef data fill:#e8f5ec,stroke:#4b8b61,color:#1d472b,stroke-width:1.5px
    classDef decision fill:#fff4df,stroke:#c18a2f,color:#563b0d,stroke-width:1.5px
    classDef cold fill:#edf3f7,stroke:#6c8799,color:#263e4d,stroke-width:1.5px
    classDef reject fill:#fff0ef,stroke:#bf6a62,color:#5b2622,stroke-width:1.5px
    classDef approve fill:#eaf6ee,stroke:#4b9566,color:#1c4b2d,stroke-width:1.5px
    classDef finish fill:#f5f5f5,stroke:#737d87,color:#26313a,stroke-width:1.3px

    class A,B,C source
    class D,J,M decision
    class E,H,L process
    class F,G,P ai
    class I data
    class K cold
    class N reject
    class O,Q approve
    class X,K1,N1,R finish
```

## How the Real Estate Workflow Works

1. **Capture the inquiry:** A buyer or renter submits their contact details and property requirements through the agency's form or a connected lead source.
2. **Validate the submission:** n8n checks that the required information is present before processing.
3. **Understand property needs:** The client intake is organized around property type, preferred area, budget, buying or renting intent, and expected timeline. The exact fields are agreed with the agency.
4. **Qualify with AI:** Groq analyzes the inquiry and returns a structured HOT, WARM or COLD category with a reason and concise summary.
5. **Save the lead:** Customer information and AI output are combined and saved in Google Sheets with a unique Lead ID and an initial Pending status.
6. **Route the lead:**
   - **HOT / WARM:** Sent for agent or manager review. The workflow waits for an approval decision.
   - **COLD:** Updated to Not Required and exits without approval or automated email.
7. **Apply the decision:**
   - **Reject:** Update the matching Lead ID to Rejected. No email is sent.
   - **Approve:** Update the matching Lead ID to Approved and continue.
8. **Send the response:** Groq drafts a personalized reply based on the inquiry. Gmail sends it after approval.

## Client Configuration

The core workflow can be reused, while each real estate agency receives its own configuration:

| Configuration | Example |
|---|---|
| Property type | Apartment, house, plot, commercial |
| Inquiry intent | Buying, renting, selling, investment |
| Preferred area | Agency's supported locations |
| Budget | Client's currency and price ranges |
| Buying / rental timeline | Immediate, within 1–3 months, researching |
| Lead source | Agency website form or agreed connected source |
| Lead database | Dedicated client Google Sheet |
| Email delivery | Client-authorized Gmail account |
| Approval owner | Assigned agent or sales manager |

The property-specific fields above are the real-estate deployment configuration. They should be added to the intake form, validation and AI prompt when adapting the reusable workflow for a client.

## Core Components

| Component | Responsibility |
|---|---|
| Lead form / connected source | Collects property inquiry details |
| n8n Webhook | Receives the inquiry |
| Validation | Checks required fields |
| Groq AI + Structured Output Parser | Classifies the inquiry and returns structured information |
| Merge | Combines lead data and AI result |
| Google Sheets | Stores each inquiry, unique Lead ID, category and status |
| Switch | Routes HOT, WARM and COLD leads |
| Wait + approval logic | Pauses HOT/WARM leads for agent review |
| Groq AI email generation | Drafts a personalized follow-up |
| Gmail | Sends the approved email |

## Lead Status and Control Rules

- Each inquiry receives a unique Lead ID. Repeat inquiries from the same contact remain separate records.
- New inquiries are saved with status **Pending**.
- Updates target the matching Lead ID, not every row with the same email address.
- HOT and WARM leads wait for a human decision.
- COLD leads are marked **Not Required** and bypass approval.
- Rejected leads are marked **Rejected** and do not trigger an email.
- Only approved leads proceed to AI email generation and Gmail delivery.

## Security and Client Isolation

- Use a separate client configuration and dedicated data destination for each agency.
- Connect each agency's own Google and email accounts using securely stored credentials.
- Never share customer records, credentials, approval tokens or webhook URLs between clients.
- Keep API credentials in n8n credentials; do not commit secrets or private customer data to this repository.
- Protect webhook and approval endpoints with authentication.
- Restrict access to client sheets and execution history.
- Test failure paths and confirm client authorization before going live.
