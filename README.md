# AgentOps Helpdesk

Multi-agent IT helpdesk built on **Microsoft Copilot Studio** and the **Power Platform**. Employees describe an IT problem in Teams; a triage agent classifies it and routes it to a specialist agent that answers from the knowledge base, raises or checks a ServiceNow ticket, or runs an approved automated fix.

> **Status: v1 in progress (rebuilt from scratch, Oct 2026).** The earlier May 2026 prototype is preserved at tag [`v0-may-2026`](../../tree/v0-may-2026).

## What it demonstrates

- **Multi-agent orchestration** in Copilot Studio: a triage router plus Knowledge, ServiceNow and Remediation specialists
- **Guardrailed automation**: the AI only proposes. Deterministic rules, human approval and a least-privilege service account decide and act
- **API-first, RPA where there is no API**: one remediation runs through Microsoft Entra ID; legacy-system fixes run through Power Automate Desktop
- **Dataverse data model** with an audit trail of every agent decision and remediation
- **ServiceNow** as the system of record: every remediation creates an incident, auto-resolved on success
- **ALM**: solution-based development, a dedicated publisher, Dev and Test environments

## Architecture

See [docs/architecture.md](docs/architecture.md) for the end-to-end flow.

## Stack

| Layer | Technology |
|---|---|
| Conversation | Microsoft Teams, Copilot Studio agents |
| Orchestration | Power Automate cloud flows |
| Legacy automation | Power Automate Desktop |
| Data | Dataverse |
| Identity and access | Microsoft Entra ID (users, departments, managers, security groups) |
| ITSM | ServiceNow |
| Knowledge | SharePoint |
| Ops console | Power Apps (model-driven) |

## Docs

- [Architecture](docs/architecture.md)
- [Data model](docs/data-model.md)
- [Design decisions](docs/decisions.md)
- [Flows](docs/flows.md)
- [Triage agent instructions](agent/triage-instructions.md)
- [Topics](agent/topics.md)
- [Platform notes](docs/platform-notes.md)

## Build plan

Built top-down, following the path a request takes.

| Step | Build | Status |
|---|---|---|
| 1 | Dataverse tables, Entra test users and groups | Done |
| 2 | Triage agent (classification) | Done |
| 3 | Log Routing Decision flow, attached to Triage | Done |
| 4 | ServiceNow agent and Create Incident flow | |
| 5 | Remediation agent, rules, approval and execution flows | |
| 6 | Knowledge agent with SharePoint | |
| 7 | IT Ops console (model-driven app) | |
| 8 | Publish to Teams, evaluation, demo | |

## Build log

| Date | Milestone |
|---|---|
| 01 Oct 2026 | Lab tenant and Dev environment, publisher `ezm`, solution, core Dataverse model, Entra test identities |
| 03 Oct 2026 | Triage agent: routing evaluation set and baseline (20/20 on GPT-5 Chat) |
| 04 Oct 2026 | Triage on Copilot Studio (generative orchestration) logging every routing decision to Dataverse through `wf_Triage_LogRoutingDecision`; identity bound to system variables; audit writes on the automation's identity ([platform notes](docs/platform-notes.md)) |
