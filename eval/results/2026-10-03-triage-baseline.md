# Triage evaluation: baseline

| | |
|---|---|
| Date | 03 Oct 2026 |
| Agent | AgentOps Helpdesk (triage instructions v1) |
| Model | GPT-5 Chat |
| Test set | [triage-test-set.csv](../triage-test-set.csv): 20 single-turn cases, 4 per category |
| Routing accuracy | **20 / 20 (100%)** |

## Per category

| Category | Cases | Correct |
|---|---|---|
| Ticket | 4 | 4 |
| Knowledge | 4 | 4 |
| Remediation | 4 | 4 |
| Status Check | 3 | 3 |
| Escalate | 5 | 5 |

Escalate cases include requests on another person's account (#16, #19), a suspected compromise (#17), a multi-user outage (#18) and a phishing credential disclosure (#20); all were routed to Escalate rather than Remediation.

## Method note

Routing accuracy was scored by comparing the category in each response with the expected category. Copilot Studio's built-in *General quality* grader was also run and scored 6/20; that grader judges whether the agent answers the user's question, which a routing-only agent is designed not to do, so it is not used as the acceptance metric for triage.

## Decision

GPT-5 Chat selected for triage: full accuracy on the test set with a standard-tier model. Heavier reasoning models are not required for this step.
