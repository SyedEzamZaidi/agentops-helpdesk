# Triage Agent — Baseline Evaluation (2026-05-07)

First evaluation run on the Triage Agent before any prompt iteration.

## Setup

- Test set: 10 questions auto-generated via Copilot Studio Quick Question Set
- Test method: General Quality (LLM-as-judge against quality rubric)
- User profile: Admin account, no tools/connectors yet

## Results

- Pass: 6/10
- Fail: 1/10
- Error: 3/10 (evaluation infrastructure errors, not agent failures)

Pass rate on tests that actually graded: 86%.

## Real failure

Input: "I have an urgent issue with my account being locked out. Can this be escalated?"

Agent response: "Escalating to a representative is not currently configured for this agent..."

Issue: Agent treated escalation as an unimplemented feature instead of a valid routing category. Fix planned in next prompt iteration.

## Suspicious pass

Input: "What's the process for requesting new software?"

Agent appeared to answer the how-to question itself rather than handing off to the Knowledge Agent. Judge passed it on quality but worth re-checking after revision.

## Errors

Three test cases didn't grade due to evaluation infrastructure errors. Will re-run before drawing conclusions.

## Next iteration

1. Fix ESCALATE behavior with explicit boundary statement
2. Verify agent isn't solving KNOWLEDGE-type questions itself
3. Re-run, target greater than 80% pass on graded tests
