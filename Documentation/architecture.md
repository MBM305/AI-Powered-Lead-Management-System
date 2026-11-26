# System Architecture

This document describes the implemented n8n lead-management workflow, from the first customer submission to each possible outcome. The diagram separates shared processing from the COLD and human-approval routes.

## End-to-end workflow

```mermaid
flowchart TB
    A(["CUSTOMER<br/>Submits inquiry"]) --> B["n8n WEBHOOK<br/>Receive lead data"]
    B --> C{"VALIDATION<br/>Are required fields present?"}

    C -->|No| X(["STOP<br/>Invalid submission"])
    C -->|Yes| D["GROQ AI CLASSIFICATION<br/>Analyze inquiry and buying intent"]
    D --> E["STRUCTURED OUTPUT<br/>Category: HOT / WARM / COLD<br/>Reason for classification"]
    E --> F["MERGE LEAD DATA<br/>Combine customer details + AI result"]
    F --> G["GENERATE UNIQUE LEAD ID<br/>Prepare initial status: Pending"]
    G --> H[("GOOGLE SHEETS<br/>Append lead record")]
    H --> I{"CLASSIFICATION ROUTER<br/>Which category was assigned?"}

    I -->|COLD| J["COLD ROUTE<br/>Update status: Not Required"]
    J --> K(["COLD COMPLETE<br/>No approval • No email"])

    I -->|HOT or WARM| L["HUMAN REVIEW<br/>Wait for approval decision"]
    L --> M{"APPROVAL DECISION"}

    M -->|Reject| N["REJECTION PATH<br/>Update matching Lead ID<br/>Status: Rejected"]
    N --> O(["REJECTED<br/>No email sent"])

    M -->|Approve| P["APPROVAL PATH<br/>Update matching Lead ID<br/>Status: Approved"]
    P --> Q["GROQ AI EMAIL GENERATION<br/>Create a personalized reply"]
    Q --> R["GMAIL DELIVERY<br/>Send email to the customer"]
    R --> S(["APPROVED & RESPONDED<br/>Email sent"])

    classDef source fill:#eaf2ff,stroke:#4778b8,color:#17365d,stroke-width:1.5px
    classDef process fill:#f4f6f8,stroke:#7b8794,color:#202b36,stroke-width:1.3px
    classDef ai fill:#f1eaff,stroke:#8560b3,color:#392052,stroke-width:1.5px
    classDef data fill:#e8f5ec,stroke:#4b8b61,color:#1d472b,stroke-width:1.5px
    classDef decision fill:#fff4df,stroke:#c18a2f,color:#563b0d,stroke-width:1.3px
    classDef cold fill:#edf3f7,stroke:#6c8799,color:#263e4d,stroke-width:1.5px
    classDef reject fill:#fff0ef,stroke:#bf6a62,color:#5b2622,stroke-width:1.5px
    classDef approve fill:#eaf6ee,stroke:#4b9566,color:#1c4b2d,stroke-width:1.5px
    classDef finish fill:#f5f5f5,stroke:#737d87,color:#26313a,stroke-width:1.3px

    class A,B source
    class C,I,M decision
    class D,E,Q ai
    class F,L process
    class G data
    class H data
    class J cold
    class N reject
    class P,R approve
    class X,K,O,S finish
```

## Workflow stages

1. **Lead reception:** The n8n Webhook receives the customer's name, email, budget and inquiry message.
2. **Validation:** Required fields are checked. Invalid submissions stop before AI classification.
3. **AI classification:** Groq analyzes the inquiry and returns a structured category (HOT, WARM or COLD) with a reason.
4. **Lead preparation:** The original customer data and AI result are combined. A unique Lead ID is generated in the workflow before the record is appended to Google Sheets.
5. **Lead record creation:** Google Sheets stores the lead with its unique Lead ID and initial `Pending` status.
6. **Category routing:**
   - **HOT / WARM:** The workflow pauses and waits for a human decision.
   - **COLD:** The record is updated to `Not Required` and finishes without approval or email.
7. **Human decision:**
   - **Reject:** The matching record is updated to `Rejected`. No email is sent.
   - **Approve:** The matching record is updated to `Approved`, then the workflow continues to email generation.
8. **Email delivery:** Groq generates a personalized reply and Gmail sends it after approval.

## Main components

| Component | Responsibility |
|---|---|
| n8n Webhook | Receives customer lead submissions |
| Validation (IF) | Checks required customer fields |
| Groq AI + Structured Output Parser | Classifies the lead and returns structured output |
| Merge | Combines original lead details with the AI result |
| Edit Fields / lead preparation | Generates the unique Lead ID and prepares lead data |
| Google Sheets | Stores each inquiry, category, Lead ID and status |
| Switch | Routes HOT, WARM and COLD leads |
| Wait + approval logic | Pauses HOT/WARM leads for human review |
| Groq AI email generation | Creates the personalized customer reply |
| Gmail | Sends the approved email |

## Data and routing rules

- Each inquiry receives its own unique Lead ID, so repeat inquiries from the same email address remain separate records.
- The Lead ID is generated in the workflow before Google Sheets appends the row.
- Google Sheets updates match the unique Lead ID, not the customer's email address.
- New lead records are stored with the initial status **Pending**.
- HOT and WARM leads wait for a human approval decision.
- COLD leads are marked **Not Required** and bypass approval.
- Rejected leads are marked **Rejected** and do not trigger an email.
- Only approved leads proceed to AI email generation and Gmail delivery.

## Security and production considerations

This diagram describes the workflow's functional path, not a complete production security architecture. Before deployment:

- Keep API credentials inside n8n credentials; never commit secrets.
- Do not publish production webhook URLs, approval tokens, Wait resume URLs or private customer data.
- Use strong authentication for externally reachable lead and approval endpoints.
- Rotate any credential or approval token that may have been exposed.
- Restrict access to connected Google Sheets and Gmail accounts.
- Add monitoring and failure notifications, and test retries and duplicate-processing behavior.
- Review privacy, retention and client access requirements before processing real customer data.

See [Security and Deployment Checklist](security-and-deployment.md) for the deployment review.
