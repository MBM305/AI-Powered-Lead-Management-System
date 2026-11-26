# Security and Deployment Checklist

This checklist separates the portfolio demonstration from the additional work required before handling a business's live customer data.

## Current project scope

The repository documents a webhook-based n8n workflow using Groq, Google Sheets, a human approval step and Gmail. It does not contain a sanitized n8n workflow export, deployment scripts, centralized monitoring or multi-client access controls.

Treat the current project as a portfolio demonstration until the checks below have been completed in the target environment.

## Before connecting a real client

### Credentials and endpoint protection

- [ ] Store Groq and Google credentials in n8n's credential manager, not in workflow fields or exported files.
- [ ] Require strong authentication for the public lead-intake webhook.
- [ ] Protect the approval endpoint independently; do not rely on an unguessable URL alone.
- [ ] Rotate any webhook secret, approval token or credential that has ever been shared in chat, screenshots, logs or public files.
- [ ] Use separate credentials and storage for each client.
- [ ] Limit access to the n8n project, Google Sheet and Gmail account to authorized people.

### Data protection

- [ ] Collect only the customer information needed for the service.
- [ ] Use synthetic or redacted information in public screenshots and demo recordings.
- [ ] Confirm the client is authorized to process the submitted customer data.
- [ ] Agree on retention, deletion and access procedures for lead records.
- [ ] Avoid placing secrets or unnecessary personal data in execution logs.

### Reliability and operational controls

- [ ] Test missing fields, malformed payloads, invalid approval decisions and provider outages.
- [ ] Confirm retry behavior cannot create duplicate lead rows or duplicate customer emails.
- [ ] Add failure notifications and a process for reviewing failed executions.
- [ ] Define who handles pending approvals and how long a lead may wait.
- [ ] Verify the workflow after changing prompts, fields, credentials or connected accounts.
- [ ] Document backup and recovery steps for workflow configuration and lead data.

### Client-specific configuration

- [ ] Map the client's lead source and required fields.
- [ ] Customize HOT / WARM / COLD criteria with the client.
- [ ] Configure the client's Google Sheet columns and access permissions.
- [ ] Customize email tone, sender identity, signature and follow-up rules.
- [ ] Test with client-approved sample data before going live.
- [ ] Obtain client approval for the final workflow and email content.

## Public repository hygiene

Before every public commit:

- Check for API keys, passwords, OAuth tokens, approval tokens, webhook URLs and Wait resume URLs.
- Check screenshots and recordings for private customer information, email addresses and browser/account details.
- Do not commit raw workflow exports unless they have been reviewed and sanitized.
- Keep real customer records and production logs out of the repository.
- Do not add a license unless you have decided how others may use, modify and redistribute this project.

## Production-readiness statement

Completing this checklist is a deployment task, not something established by the portfolio screenshots or demo video. Validate each item in the actual n8n environment before describing a client deployment as production-ready.
