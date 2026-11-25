# System Architecture

This document describes the end-to-end flow of the AI-Powered Lead Management System, from lead submission through AI classification, human review, and customer email delivery.

## Architecture Diagram

```mermaid
flowchart TB
    subgraph INTAKE["01 · Lead Intake"]
        A(["Customer submits inquiry"]) --> B["n8n Webhook"]
        B --> C["Validate required fields"]
        C -->|Invalid| X(["Stop · Invalid submission"])
    end

    subgraph AI["02 · AI Analysis"]
        C -->|Valid| D["Groq AI classification"]
        D --> E["Structured result<br/>HOT · WARM · COLD"]
        E --> F["Merge lead data + AI result"]
    end

    subgraph DATA["03 · Lead Record"]
        F --> G[("Google Sheets<br/>Create lead record")]
        G --- H["Unique Lead ID<br/>for each inquiry"]
    end

    subgraph ROUTING["04 · Decision & Routing"]
        G --> I{"Lead category"}
        I -->|HOT / WARM| J["Status: Pending"]
        I -->|COLD| K["Status: Not Required"]
        J --> L["Wait for human decision"]
        L --> M{"Approved?"}
        M -->|No| N["Update record: Rejected"]
        M -->|Yes| O["Update record: Approved"]
    end

    subgraph DELIVERY["05 · Approved Lead Follow-up"]
        O --> P["Groq AI generates<br/>personalized email"]
        P --> Q["Gmail sends email"]
        Q --> R(["Completed"])
    end

    K --> S(["Completed · No email"])
    N --> T(["Completed · No email"])

    classDef process fill:#eef4ff,stroke:#4776b8,color:#172b4d,stroke-width:1.4px
    classDef ai fill:#f2eaff,stroke:#8a63c7,color:#352052,stroke-width:1.4px
    classDef data fill:#e8f5ee,stroke:#438b65,color:#173c29,stroke-width:1.4px
    classDef decision fill:#fff5df,stroke:#c28a2c,color:#50370d,stroke-width:1.4px
    classDef terminal fill:#f1f3f5,stroke:#7b8794,color:#263238,stroke-width:1.2px
    class B,C,F,J,L,O,N,P,Q process
    class D,E ai
    class G,H data
    class I,M decision
    class X,R,S,T terminal
```

## Process Breakdown

| Stage | What happens |
|---|---|
| Lead intake | The webhook receives a customer inquiry and required fields are validated. |
| AI analysis | Groq classifies the inquiry as HOT, WARM, or COLD and returns a structured result with a reason. |
| Lead record | The lead and AI result are combined and stored in Google Sheets. Each inquiry receives a unique Lead ID. |
| Routing | HOT and WARM leads enter human approval. COLD leads are marked `Not Required` and bypass approval. |
| Human decision | Approval or rejection updates the correct row using the unique Lead ID. |
| Email delivery | Only approved leads proceed to AI email generation and Gmail delivery. Rejected and COLD leads do not receive this automated email. |

## Technology Components

| Component | Role |
|---|---|
| n8n | Orchestrates the complete workflow |
| Webhook | Receives lead submissions and approval decisions |
| Groq AI | Performs lead classification and personalized email generation |
| Structured Output Parser | Structures the classification response |
| Google Sheets | Stores lead details, category, and status |
| Wait and approval logic | Keeps HOT and WARM leads pending for human review |
| Gmail | Delivers the email after approval |

## Key Design Decisions

- **Human-in-the-loop:** AI classifies and drafts; a person makes the approval decision.
- **Unique record matching:** Updates use the Lead ID, so multiple inquiries from one email address remain separate.
- **Controlled email delivery:** The automated email path is available only after approval.
- **Clear outcomes:** Rejected and COLD leads finish without the approval email.

## Security Considerations

- Store API credentials in n8n credentials; never commit secrets to this repository.
- Do not publish production webhook URLs, approval tokens, or private customer data.
- Protect approval endpoints with authentication and rotate any credential that may have been exposed.
- Restrict access to connected Google Sheets and Gmail accounts.
- Review execution logs and test failure paths before production use.
