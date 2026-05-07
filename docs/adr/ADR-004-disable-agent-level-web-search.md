# ADR-004: Disable Agent-Level Web Search for Router Agents
- Status: Accepted
- Date: 08-May-2026
- Author: Syed Ezam Zaidi
## Context
Copilot Studio exposes a setting called "Web Search" at the agent level. When enabled, the agent can search public websites via Bing and use the results to compose responses. The setting is on by default for new agents created in some Copilot Studio configurations.

During Day 3 evaluation, after the Escalate topic was disabled, eval results showed 4 pass / 6 fail with an unusual pattern: passing responses contained full IT helpdesk guides with citations from public university support sites (support.oakland.edu) and consulting blogs (masonadvisory.com). The Triage Agent has no Knowledge sources configured and is built purely for classification and routing. The cited content was not coming from the agent's instructions or any configured grounding source.

Investigation traced the responses to agent-level Web Search being enabled. The generative orchestrator was bypassing the Triage routing prompt entirely on certain queries, deciding instead to answer the user directly from web results. The "passing" tests were silent architectural failures — the agent was not classifying or routing, it was answering helpdesk questions with hallucinated authority by stitching together random university IT support pages.

After Web Search was disabled (combined with the ADR-003 fix to allow ungrounded responses), the Triage Agent returned 10/10 architecturally correct responses, classifying every query and routing to the appropriate downstream agent without attempting to answer.

This is the inverse failure mode of ADR-001: where ADR-001 covered topics overriding prompt behavior, ADR-004 covers a built-in capability overriding prompt behavior. Same lesson, different surface.
## Decision
Agent-level Web Search is OFF for all router-style agents in the AgentOps Helpdesk system.

Specifically:
- **Triage Agent** — OFF. Confirmed Day 3.
- **ServiceNow Agent** — OFF. Responses must come from ServiceNow API tool calls, not the public web.
- **Remediation Agent** — OFF. Actions are gated by tool calls; web context is irrelevant and dangerous.
- **Escalation Agent** — OFF. Handoff context derives from conversation state and ticket data, not the web.
- **Knowledge Agent** (Week 2-3) — Decision deferred. Likely OFF (grounded in Azure AI Search / Foundry RAG instead), but Web Search may have a narrow legitimate use as a last-resort fallback. Decision lands when Knowledge Agent is designed.

The default stance for any new agent in this project is Web Search OFF unless an explicit architectural reason requires it.
## Reasons
- Web Search at the agent level overrides the routing prompt. The model decides "I can answer this from the web" before deciding "I should classify and route." For a router, this is categorically wrong behavior.
- The capability creates silent architectural failures that look like passes in evaluation. The Day 3 eval showed 4/10 "passing" when the agent was in fact failing 10/10 architecturally — the grader saw coherent helpdesk answers and approved them, while the agent was bypassing its actual job.
- Citations from public web pages give a false impression of authority. Recruiters or exam reviewers reading the eval output would see the agent inventing helpdesk procedures from random universities and consultancies — an immediately disqualifying behavior for a portfolio project.
- For specialist agents, web data is structurally inappropriate. A ServiceNow Agent must speak from ServiceNow data; a Remediation Agent must speak from approved actions; a Knowledge Agent must speak from approved internal documentation. Web search opens a door none of them should walk through.
## Alternatives considered
- Leave Web Search ON and rely on prompt instructions to discourage its use. Rejected: the Day 3 eval proved the orchestrator's decision to invoke Web Search happens before the routing prompt is consulted. Prompt-level constraints cannot reliably override a built-in capability.
- Leave Web Search ON for "general helpfulness." Rejected: there is no scenario in this multi-agent system where the public web is the right source. Every legitimate response path is either instruction-driven (router) or grounded in an approved source (specialist).
- Remove Web Search entirely from the project as a blanket policy. Rejected: a future Knowledge Agent variant or research-style agent might legitimately use it. Per-agent decisions are the right granularity.
## Consequences
- Each new agent's spec must explicitly state the value of this setting and the reason, alongside the ADR-003 ungrounded-responses setting. These two settings together define the agent's response surface and must be documented per-agent.
- Eval interpretation changes. Without Web Search masking, eval results now reflect actual agent behavior. The Day 2 baseline (6/1/3) is retroactively understood to have been Web Search-inflated and is no longer a valid reference point.
- The Day 3 eval result (10/10 architecturally correct, 8/10 by auto-grader) becomes the new baseline for the Triage Agent. The 2 grader "fails" are themselves a finding documented in the iteration-2 eval report — auto-graders do not understand router-style agents and will misclassify correct handoff behavior as failure.
- This ADR pairs with ADR-003 as the two-setting pattern that defines a router agent: ungrounded ON + Web Search OFF. The two-setting pattern for grounded specialists is the inverse: ungrounded OFF + Web Search OFF (with the Knowledge Agent exception still TBD).
## When to revisit
- When the Knowledge Agent is designed and the question of "last-resort fallback to web" is decided.
- When a future agent in the project legitimately needs web context (e.g., a research or news-summarization agent — none currently planned).
- When any future eval shows the agent ignoring or bypassing its instructions and the cause is traced to a built-in capability — same diagnostic playbook applies.
