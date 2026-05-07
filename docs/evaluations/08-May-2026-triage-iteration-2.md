# Triage Agent — Evaluation Iteration 2
- Date: 08-May-2026
- Author: Syed Ezam Zaidi
- Agent: AgentOps Triage
- Bot ID: 34084b5f-3749-f111-bec7-7ced8d9ec815
- Model: Claude Sonnet 4.6
## Summary
- **Auto-grader result:** 8 pass / 2 fail (80%)
- **Architectural review:** 10 pass / 0 fail (100%)
- **Status:** First clean signal on actual prompt performance. Both grader "fails" are correct routing behavior misclassified by a generic helpfulness rubric.

## Configuration changes since baseline
Three changes between Day 2 baseline (6 pass / 1 fail / 3 errors) and this run:

1. **Escalate System Topic disabled.** Per ADR-002, the Escalate topic was identified as overriding the routing prompt with default Microsoft copy. Disabling it removed the override. Re-enable + Power Automate flow trigger deferred to Week 2.
2. **Web Search disabled at agent level.** Per ADR-004, agent-level Web Search was found to be the actual source of "passing" responses in the Day 2 baseline and the first part of Day 3. The agent was bypassing the routing prompt and answering helpdesk questions directly from public web pages, with citations to random university IT support sites. Disabling Web Search forced the agent to actually use its instructions.
3. **Allow ungrounded responses set to ON.** Per ADR-003, the Day 2 plan to flip this OFF was incorrect for a router agent. The Triage Agent has no Knowledge sources by design — its response path is purely instruction-driven. With ungrounded responses blocked, the agent cannot produce a routing decision and falls through to Fallback. ON is the correct value for instruction-driven routers.

## Test set
The same auto-generated 10-question set from the Day 2 baseline was reused. Reusing the test set holds the input variable constant so any change in result attributes to configuration changes, not test variance.

Note on auto-generation: a fresh attempt at auto-generation during Day 3 was blocked by Azure Responsible AI content filter. This is an architectural finding in itself — the same RAI configuration applied to user-facing inference is also applied to internal test generation, with no per-context tuning. Foundry's per-deployment content filter configuration is the right surface for solving this in Week 3 (BYO-model decision is partially justified by this finding).

## Results
| # | Question | Response | Grader | Architectural |
|---|---|---|---|---|
| 1 | "I can't remember my username for the VPN. Who can help me with that?" | Routed to remediation team for credentials recovery | Pass | Pass |
| 2 | "Is there an update on my request for a new monitor?" | Asked for ticket/request number to look up status | Pass | Pass |
| 3 | "My laptop keeps freezing and I can't get any work done. What should I do?" | Routed to ticketing team with internal category tagging | Pass | Pass |
| 4 | "I need access to the finance shared drive. Can you help?" | Routed to remediation team with note on access level + manager approval | Pass | Pass |
| 5 | "What's the process for requesting new software?" | Routed to knowledge agent for procedural how-to | **Fail** | **Pass** |
| 6 | "I submitted a ticket last week but haven't heard back. Can you check on it?" | Asked for ticket number to retrieve details | Pass | Pass |
| 7 | "I have an urgent issue with my account being locked out. Can this be escalated?" | Routed to escalation team for prioritized handling | Pass | Pass |
| 8 | "I need to install Adobe Acrobat on my laptop. Who can assist with that?" | Routed to remediation team for software install request | Pass | Pass |
| 9 | "How do I set up my work email on my phone?" | Routed to knowledge agent for setup walkthrough | **Fail** | **Pass** |
| 10 | "Can you help me check the status of my open IT ticket?" | Asked for ticket number to retrieve status | Pass | Pass |

## Why grader and architectural review disagree
Two responses (#5 and #9) were marked Fail by the auto-grader. Both are correct routing behavior:

- #5 response: "Sounds like a how-to question on software request procedures. Let me hand you to our knowledge agent who can walk you through the exact process."
- #9 response: "Sounds like a how-to question. Let me hand you to our knowledge agent who can walk you through the steps to set up your work email on your phone."

These are textbook Triage Agent outputs. The agent classified the intent (knowledge / how-to), identified the correct downstream specialist (knowledge agent), and handed off cleanly without attempting to answer the question itself. That is the design.

The auto-grader scores against a generic helpfulness rubric: "Did the response answer the user's question?" For a single-agent chatbot, that rubric is valid. For a router in a multi-agent system, the correct behavior — refusing to answer and handing off — looks like failure to a grader that does not understand the architecture.

The grader is structurally wrong for this agent type. The agent is correct.

## Architectural finding: auto-graders are role-blind
This is the first significant finding for the Foundry custom evaluation harness work in Week 3. The built-in Copilot Studio eval is fine for smoke tests but cannot grade multi-agent routing behavior. A custom eval harness needs role-aware metrics:

- For router agents: Did the router correctly identify the target specialist? Did the router refuse to answer the question itself? Did the handoff include the right context for the downstream agent?
- For grounded specialist agents: Was the response grounded in the expected source? Did the agent refuse when retrieval failed?
- For action-gated agents: Did the agent invoke the correct tool? Did the agent require approval where required?

A single "did this answer the user" metric cannot evaluate any of these correctly.

This finding promotes the Foundry custom eval harness from a Week 3 nice-to-have to a Week 3 architectural necessity. The justification is now in the project's own evaluation history, not just theoretical CV-padding.

## Comparison to Day 2 baseline
The Day 2 baseline (6 pass / 1 fail / 3 errors, 86% on graded) is no longer a valid reference point. That run's "passes" were Web Search responses bypassing the routing prompt, not the prompt actually working. The errors in that run were Escalate topic overrides.

This iteration is the first eval where the prompt is actually in the response path with no masking layers. The 10/10 architectural result reflects real prompt performance.

## Open items
- Re-enable Escalate topic with Power Automate flow trigger (Week 2, per ADR-002).
- Build Foundry custom eval harness with role-aware metrics (Week 3).
- Add edge-case tests to the eval set for system topic coverage — Fallback, On Error, Sign in. Current 10-question set covers happy-path routing only.
- Investigate auto-generation RAI block — confirm Foundry BYO-model with custom content filter resolves it (Week 3).

## Configuration snapshot at time of run
- Generative AI orchestration: ON
- Allow ungrounded responses: ON (per ADR-003)
- Web Search: OFF (per ADR-004)
- Knowledge sources attached: 0
- Tools attached: 0
- Custom topics: 4 (no triggers — present for documentation only)
- System topics: 9 (Escalate disabled, others default — per ADR-002)
