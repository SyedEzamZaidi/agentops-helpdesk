# Triage regression run: 20-case set, updated instructions

**Date:** 04 Oct 2026 · **Agent:** AgentOps Helpdesk (Standard), GPT-5 Chat · **Instructions:** [agent/triage-instructions.md](../../agent/triage-instructions.md) (04 Oct version)
**Test set:** [triage-test-set-single.csv](../triage-test-set-single.csv) · **Raw export:** [runs/2026-10-04-1814-triage-standard-20case.csv](../runs/2026-10-04-1814-triage-standard-20case.csv)

## Results

| Check | Result |
|---|---|
| Cases executed | 14 / 20 (6 throttled: `GenAIToolPlannerRateLimitReached`) |
| Timeline / outcome language ("shortly", "soon") | **0 / 14** (smoke run before the instruction change: 4 / 5) |
| Internal labels in reply text | 0 / 14 |
| Raw tool arguments echoed to the user | **2 / 14**: JSON with `text`, `text_2`, `text_3`, `explanation_of_tool_call` printed above the reply |
| Escalate cases captured by the system *Escalate* topic | **2 / 4** executed Escalate cases ("my account has been hacked", "clicked a link … entered my password"): generic "Escalating to a representative is not currently configured" reply, logging tool not called |
| Category accuracy | Pending: scored from Agent Decision rows joined on Conversation ID |

*Answer quality* is reported by the export but is not an acceptance metric for a router: it fails replies that log a request instead of answering it, which is the intended behaviour.

## Findings

1. **Reply policy fix confirmed.** Removing the old reply block and adding the explicit no-timeline rule eliminated timeline promises and label leaks.
2. **Tool-argument echo (new).** On 2 cases the model printed the tool call's arguments, including internal keys, before its reply. Internal data reaching the user is a policy failure even when the classification is right.
3. **Security incidents bypass logging (critical).** The built-in *Escalate* system topic is triggered by escalation intent and pre-empts generative orchestration. The two most security-sensitive cases got a placeholder reply and **no Agent Decision row**, so they leave no audit trail.

## Actions

1. Add to the instructions: never output JSON, tool names or tool parameters to the user.
2. Redesign the system *Escalate* topic: log the request with Category = Escalate through the same tool, then hand over with a clear message (security incidents get the IT security contact). Add these cases to the regression set with the expected tool.
3. Rerun the 6 throttled cases in a separate batch, then score category accuracy from Agent Decisions.

## Rerun after fixes

Fixes applied: actions 1 and 2 ([instructions](../../agent/triage-instructions.md), [Escalate topic](../../agent/topics.md)). Graders: *Tool use* (expected tool set per case) and *Reply policy check* (custom); *Answer quality* removed.

| Batch | Cases | Tool use | Reply policy | Raw export |
|---|---|---|---|---|
| 1: throttled cases | 5 | n/a (Answer quality only) | No labels, JSON or timelines found on review | [batch 1](../runs/2026-10-04-1830-triage-rerun-batch1.csv) |
| 2: Escalate + remaining throttled | 5 | **5 / 5** | **5 / 5** | [batch 2](../runs/2026-10-04-1912-triage-rerun-batch2.csv) |

- Both security incidents ("my account has been hacked", "clicked a link … entered my password") now call the logging tool and tell the user to change their password and contact IT security.
- Requests about another person's account (Mark's unlock, manager's MFA) are logged and escalated without promising the change.
- All 20 cases have now executed.

## Category accuracy (from Dataverse)

Each executed case was joined to its Agent Decision row on Conversation ID; the logged choice value was compared with the expected category. Per-case sheet: [2026-10-04-triage-category-scoring.csv](2026-10-04-triage-category-scoring.csv).

| Category | Correct |
|---|---|
| Ticket | 4 / 4 |
| Knowledge | 4 / 4 |
| Remediation | 4 / 4 |
| Status Check | 3 / 3 |
| Escalate | 5 / 5 |
| **Total** | **20 / 20** |

- Every case produced exactly one Agent Decision row (20 / 20 logged), with Requester Email from the signed-in user and the Conversation ID set.
- Matches the routing baseline from 03 Oct (20 / 20), now measured on what the agent logged rather than what it printed.
