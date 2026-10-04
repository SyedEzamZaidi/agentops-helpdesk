# Flows

## wf_Triage_LogRoutingDecision

Agent flow called by the Triage agent once per request, right after classification. Writes one Agent Decision row and returns its ID so later steps (ServiceNow, Remediation) can update or link to the same row.

**Tool on the agent:** `wf_Triage_LogRoutingDecision_v2` · credentials: maker-provided · connection runtime: embedded (see [D17](decisions.md#d17-audit-writes-run-on-the-automations-identity-not-the-users), [D18](decisions.md#d18-verify-connection-runtime-mode-in-the-solution-before-every-deployment)).

### Inputs

| Input | Type | Filled by | Internal key |
|---|---|---|---|
| RequestSummary | Text | AI | `text` |
| Category | Text, fixed list: Knowledge, Ticket, Remediation, Status Check, Escalate | AI | `text_2` |
| RoutingReason | Text | AI | `text_3` |
| RequesterEmail | Text | `System.User.Email` | `text_4` |
| ConversationId | Text | `System.Conversation.Id` | `text_1` |

Internal keys follow creation order, not display order; expressions reference the key.

### Steps

1. **When an agent calls the flow**
2. **Dataverse – Add a new row** (Agent Decisions)

   | Column | Value |
   |---|---|
   | Name | RequestSummary |
   | Request Summary | RequestSummary |
   | Category | Category label mapped to the choice value (below) |
   | Routing Reason | RoutingReason |
   | Requester Email | RequesterEmail |
   | Conversation ID | ConversationId |
   | Handled By | Triage |

3. **Respond to the agent** → `DecisionId` (Text) = the new row's `ezm_agentdecisionid`

### Category mapping

| Label | Choice value |
|---|---|
| Knowledge | 791760000 |
| Ticket | 791760001 |
| Remediation | 791760002 |
| Status Check | 791760003 |
| Escalate (and any unexpected value) | 791760004 |

```
if(equals(triggerBody()?['text_2'],'Knowledge'),791760000,
if(equals(triggerBody()?['text_2'],'Ticket'),791760001,
if(equals(triggerBody()?['text_2'],'Remediation'),791760002,
if(equals(triggerBody()?['text_2'],'Status Check'),791760003,791760004))))
```

Unknown values fall through to Escalate, so a bad classification is routed to a human rather than dropped.
