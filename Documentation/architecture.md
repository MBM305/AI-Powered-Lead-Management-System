# System Architecture

This document explains the complete lead-management workflow, from the first customer submission to each possible outcome. The diagram separates the common processing stages from the COLD and human-approval routes so the decisions and connections are easy to trace.

## End-to-End Workflow

```mermaid
flowchart TB
    A(["CUSTOMER<br/>Submits inquiry"]) --> B["n8n WEBHOOK<br/>Receive lead data"]
    B --> C{"VALIDATION<br/>Are required fields present?"}

    C -->|No| X(["STOP<br/>Invalid submission"])
    C -->|Yes| D["GROQ AI CLASSIFICATION<br/>Analyze inquiry and buying intent"]
    D --> E["STRUCTURED OUTPUT<br/>Category: HOT / WARM / COLD<br/>Reason for classification"]
    E --> F["MERGE LEAD DATA<br/>Combine customer details + AI result"]
    F --> G[("GOOGLE SHEETS<br/>Append a new lead record<br/>Generate unique Lead ID<br/>Initial status: Pending")]
    G --> H{"CLASSIFICATION ROUTER<br/>Which category was assigned?"}

    H -->|COLD| I["COLD ROUTE<br/>Update status: Not Required"]
    I --> J(["COLD COMPLETE<br/>No approval • No email"])

    H -->|HOT or WARM| K["HUMAN REVIEW<br/>Wait for approval decision"]
    K --> L{"APPROVAL DECISION"}

    L -->|Reject| M["REJECTION PATH<br/>Update matching Lead ID<br/>Status: Rejected"]
    M --> N(["REJECTED<br/>No email sent"])

    L -->|Approve| O["APPROVAL PATH<br/>Update matching Lead ID<br/>Status: Approved"]
    O --> P["GROQ AI EMAIL GENERATION<br/>Create a personalized reply"]
    P --> Q["GMAIL DELIVERY<br/>Send email to the customer"]
    Q --> R(["APPROVED & RESPONDED<br/>Email sent"])

    classDef source fill:#eaf2ff,stroke:#4778b8,color:#17365d,stroke-width:1.5px
    classDef process fill:#f4f6f8,stroke:#7b8794,color:#202b36,stroke-width:1.3px
    classDef ai fill:#f1eaff,stroke:#8560b3,color:#392052,stroke-width:1.5px
    classDef data fill:#e8f5ec,stroke:#4b8b61,color:#1d472b,stroke-width:1.5px
    classDef decision fill:#fff4df,stroke:#c18a2f,color:#563b0d,stroke-width:1.5px
    classDef cold fill:#edf3f7,stroke:#6c8799,color:#263e4d,stroke-width:1.5px
    classDef reject fill:#fff0ef,stroke:#bf6a62,color:#5b2622,stroke-width:1.5px
    classDef approve fill:#eaf6ee,stroke:#4b9566,color:#1c4b2d,stroke-width:1.5px
    classDef finish fill:#f5f5f5,stroke:#737d87,color:#26313a,stroke-width:1.3px

    class A,B source
    class C,H,L decision
    class D,E,P ai
    class F,K process
    class G data
    class I cold
    class M reject
    class O,Q approve
    class X,J,N,R finish
```

## Workflow Stages

1. **Lead reception:** The n8n Webhook receives the customer's name, email, budget and inquiry message.
2. **Validation:** Required fields are checked. Invalid submissions stop before AI classification.
3. **AI classification:** Groq analyzes the inquiry and returns a structured category (HOT, WARM or COLD) with a reason.
4. **Lead record creation:** Customer data and AI output are combined. Google Sheets stores the lead with a unique Lead ID and an initial Pending status.
5. **Category routing:**
   - **HOT / WARM:** The workflow pauses and waits for a human decision.
   - **COLD:** The record is updated to Not Required and finishes without approval or email.
6. **Human decision:**
   - **Reject:** The matching record is updated to Rejected. No email is sent.
   - **Approve:** The matching record is updated to Approved, then the workflow continues to email generation.
7. **Email delivery:** Groq generates a personalized reply and Gmail sends it to the customer after approval.

## Main Components

| Component | Responsibility |
|---|---|
| n8n Webhook | Receives customer lead submissions |
| Validation (IF) | Checks required customer fields |
| Groq AI + Structured Output Parser | Classifies the lead and returns structured output |
| Merge | Combines original lead details with the AI result |
| Google Sheets | Stores each inquiry, unique Lead ID, category and status |
| Switch | Routes HOT, WARM and COLD leads |
| Wait + approval logic | Pauses HOT/WARM leads for human review |
| Groq AI email generation | Creates the personalized customer reply |
| Gmail | Sends the approved email |

## Data and Routing Rules

- Each inquiry receives its own unique Lead ID, so repeat inquiries from the same email address remain separate records.
- Google Sheets is updated using the unique Lead ID, not the customer's email address.
- New lead records are stored with the initial status **Pending**.
- HOT and WARM leads wait for a human approval decision.
- COLD leads are marked **Not Required** and bypass approval.
- Rejected leads are marked **Rejected** and do not trigger an email.
- Only approved leads proceed to AI email generation and Gmail delivery.

## Security Considerations

- Keep API credentials inside n8n credentials; never commit secrets to this repository.
- Do not publish production webhook URLs, approval tokens, resume URLs or private customer data.
- Protect approval endpoints with authentication and rotate any credential that may have been exposed.
- Restrict access to connected Google Sheets and Gmail accounts.
- Review execution logs and test failure paths before production use.
