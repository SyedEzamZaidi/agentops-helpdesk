# Data model (Dataverse)

Publisher prefix: `ezm`. All tables live in the **AgentOps Helpdesk** solution.

## Global choice

**Request Category** (`ezm_requestcategory`): Knowledge, Ticket, Remediation, Status Check, Escalate. Shared by Agent Decision and Eval Case.

## Agent Decision (`ezm_agentdecision`)

One row per request routed by the Triage agent. Ownership: user or team. Auditing on.

| Column | Type | Notes |
|---|---|---|
| Name (primary) | Text | Built by the flow: category plus the first 50 characters of the message |
| User Message | Multiple lines | Raw text from the employee |
| Category | Choice (Request Category) | Triage decision |
| Routing Reason | Multiple lines | Agent's short explanation, used for debugging and evaluation |
| Handled By | Choice | Triage, Knowledge, ServiceNow, Remediation, Escalation |
| Requester Email | Email | From the Teams sign-in |
| Conversation ID | Text | Correlates rows from the same chat |
| Incident Number | Text | ServiceNow key, if a ticket was created |
| Outcome | Choice | Answered, Ticket Created, Remediated, Escalated, Failed |

## Remediation Catalog (`ezm_remediationcatalog`)

Approved automated fixes; the agent can only run what is listed here. Ownership: organization (reference data). Auditing on.

| Column | Type | Notes |
|---|---|---|
| Fix Name (primary) | Text | Human-readable |
| Description | Multiple lines | When the fix applies |
| Action Code | Choice | UNLOCK_ACCOUNT, RESET_MFA, ENABLE_VPN_ACCESS. Stable machine key |
| Risk Level | Choice | Low, Medium, High |
| Execution Channel | Choice | API, Desktop Flow |
| Requires Approval | Yes/No | Default Yes (secure by default) |
| Self Service Only | Yes/No | Default Yes: the target must be the requester |
| Active | Yes/No | Soft delete |
| Eligible Departments | Multiple lines | Comma-separated Entra departments; blank means all |

Seed data:

| Fix | Code | Risk | Channel | Approval | Eligible |
|---|---|---|---|---|---|
| Unlock user account | UNLOCK_ACCOUNT | Low | Desktop Flow | No | All |
| Enable VPN access | ENABLE_VPN_ACCESS | Medium | API | Yes | Engineering, IT |
| Reset MFA | RESET_MFA | High | Desktop Flow | Yes | All |

## Remediation Request (`ezm_remediationrequest`)

Each attempt to run a catalog fix. Ownership: user or team. Auditing on.

| Column | Type | Notes |
|---|---|---|
| Request Number (primary) | Autonumber | `REM-00001` |
| Fix | Lookup to Remediation Catalog | |
| Agent Decision | Lookup to Agent Decision | Links back to the conversation |
| Requester Email | Email | Verified identity |
| Target User Email | Email | Whose account is changed; must match the requester when Self Service Only |
| Request Status | Choice | Pending Approval (default), Approved, Rejected, Running, Succeeded, Failed |
| Approver | Text | |
| Approver Comments | Multiple lines | |
| Decision Date | Date and time | User local |
| Incident Number | Text | ServiceNow key |
| Run Result | Multiple lines | API or desktop flow output, or the error |

## Planned

- **Eval Case** and **Eval Result** tables for routing-accuracy evaluation.

## Data that lives outside Dataverse

| Data | Source of truth |
|---|---|
| Users, departments, managers | Microsoft Entra ID |
| VPN access | Membership of the `VPN-Users` security group |
| Incidents | ServiceNow (stored here only as an external key) |
| Knowledge articles | SharePoint |
