# ADR-002: System Topic Audit (Triage Agent)
- Status: Accepted
- Date: 08-May-2026
- Author: Syed Ezam Zaidi
## Context
The Triage Agent is built on Copilot Studio with 9 auto-created System Topics. ADR-001 already established that auto-created topics can silently override prompt-driven behavior, after the Escalate topic was found intercepting LLM-bound classifications during Day 2 evaluation.

With no eval coverage on edge cases, there was a real risk that other system topics carried similar landmines — default Microsoft messages or behaviors that could silently interfere with intended LLM-orchestrated routing.

Initial instinct was to defer the audit to Week 2, when Power Automate flows and integrations land. That position was reversed: Escalate had already demonstrated the failure mode, and waiting for Week 2 meant operating an agent with 8 unaudited interrupt paths during the most active prompt-iteration phase.

A 20-minute defensive smoke audit was performed — opening each system topic, reading the Message nodes, and flagging anything likely to interfere with current routing behavior.
## Decision
All 9 system topics audited. The Escalate topic is disabled. The remaining 8 topics retain their default Microsoft configuration. Action items are captured for future iterations when downstream context (integrations, persona pass, telemetry) lands.

Per-topic verdicts:
- **Conversation Start** (On Conversation Start) — Leave enabled, default. Action: persona pass before LinkedIn pivot.
- **Conversational boosting** (On Unknown Intent) — Leave enabled, default. Action: decide RAG strategy in Week 3 (Foundry vs built-in).
- **End of Conversation** (On Redirect) — Leave enabled, default. Action: wire up when first specialist agent ships.
- **Escalate** (On Talk to Agent) — Disabled. Action: re-enable + Power Automate flow trigger in Week 2.
- **Fallback** (On Unknown Intent) — Leave enabled, default. Action: revisit after specialists added.
- **Multiple Topics Matched** (On Select Intent) — Leave enabled, default. Action: revisit if trigger phrases added to custom topics.
- **On Error** (On Error) — Leave enabled, default. Action: confirm App Insights picks up the Log custom telemetry event node (Week 2-3).
- **Reset Conversation** (On Redirect) — Leave enabled, default. Action: none.
- **Sign in** (On Sign In) — Leave enabled, default. Action: customize message when ServiceNow auth configured (Week 2).
## Reasons
- Eval-driven, not speculative. The audit was triggered by Escalate's proven interference, not by hypothetical concerns. Only one topic (Escalate) is altered from default; the other 8 are unchanged because there is no current evidence they interfere.
- YAGNI applied at the topic level. No customization for use cases that don't exist yet (telemetry, auth flows, persona). Action items are deferred with explicit triggers, not vague intent.
- Defensive minimum. A 20-minute smoke read of all 8 remaining topics is cheap insurance against another Escalate-style surprise mid-eval.
- Architectural insight surfaced. Conversational boosting was confirmed to be the Knowledge-tab-driven Generative answers pipeline, NOT generative orchestration. This affects the RAG strategy decision in Week 3 — adding any data sources to the Knowledge tab will activate this topic and could override the routing prompt.
## Alternatives considered
- Full audit deferred to Week 2. Rejected: Escalate had already proven the failure mode; deferring meant running 8 unaudited interrupt paths during the most active prompt-iteration phase of the project.
- Disable all unused system topics. Rejected: system topics encode default conversational behaviors (error handling, reset, sign-in) that we will need later; disabling them now creates silent gaps that will be hard to diagnose in Week 2.
- Customize all message nodes preemptively for the AgentOps Helpdesk persona. Rejected as YAGNI — no current eval requires customized copy, and persona pass is better done once before the LinkedIn pivot.
## Consequences
- Triage Agent runs with 8 default-configured system topics + 1 disabled (Escalate). Default Microsoft messages may surface in edge cases (Fallback, Multiple Topics Matched) but do not interfere with the current routing test set.
- Conversational boosting is now a known landmine for the Knowledge Agent design. Week 3 RAG strategy must explicitly decide whether to use this built-in pipeline or build RAG via Foundry Prompt Flow.
- Action items captured per topic become inputs to the Week 2 integration work — particularly Sign in (ServiceNow auth) and On Error (App Insights telemetry).
- The audit method itself becomes the standard playbook for new agents added later in the project.
## When to revisit
- When the Knowledge Agent is built and RAG strategy is finalized — Conversational boosting decision lands here.
- When ServiceNow integration adds end-user authentication — Sign in topic message gets customized.
- When App Insights is wired up — On Error telemetry event flow is verified.
- When persona / brand pass happens before LinkedIn pivot — Conversation Start customization lands here.
- When any future eval shows a system topic interfering with intended behavior — that topic gets edited or disabled, regardless of plan.
