# Triage smoke run: Standard agent with logging tool

**Date:** 04 Oct 2026 · **Agent:** AgentOps Helpdesk (Standard), generative orchestration, GPT-5 Chat · **Tool:** `wf_Triage_LogRoutingDecision_v2`
**Raw export:** [runs/2026-10-04-1802-triage-standard-smoke.csv](../runs/2026-10-04-1802-triage-standard-smoke.csv)

Smoke run on Copilot Studio's generated question set (11 single-turn cases) to validate the end-to-end path (classification → tool call → Dataverse row → user reply) before the full 20-case regression run. Not a baseline.

## Results

| Check | Result |
|---|---|
| Cases executed | 5 / 11 (6 throttled: `GenAIToolPlannerRateLimitReached`) |
| Tool use (graded cases) | 2 / 2 pass; expected tool set on 2 cases only |
| Logging tool called (executed cases) | 5 / 5 (replies confirm logging; rows in Agent Decisions) |
| Internal labels shown to user | 0 / 5 (previous run leaked `Category / Summary / Reason` on 1 case) |
| Timeline promises ("shortly") | 4 / 5: violates the reply policy |

## Actions

1. **Reply policy:** add an explicit rule against time language ("shortly", "soon") to the agent instructions; enforce with the *Reply policy check* custom grader.
2. **Throttling:** run evaluations in batches of 5 with a pause between batches.
3. **Coverage:** run the fixed [20-case set](../triage-test-set-single.csv) with the expected tool set on every case; score category accuracy from Agent Decision rows joined on Conversation ID.
