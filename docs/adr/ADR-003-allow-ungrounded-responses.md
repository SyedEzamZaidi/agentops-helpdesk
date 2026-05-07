# ADR-003: Allow Ungrounded Responses (Per-Agent Setting)
- Status: Accepted
- Date: 08-May-2026
- Author: Syed Ezam Zaidi
## Context
Copilot Studio agents have a setting called "Allow ungrounded responses." When ON, the agent can respond using the model's general knowledge and the instructions field alone. When OFF, the agent is restricted to responses grounded in attached Knowledge sources or Tools — and must refuse if no grounding source is available.

During Day 3 evaluation, the Triage Agent returned 0/10 pass after this setting was flipped OFF. Every response was the Fallback topic's default message ("I'm sorry, I'm not sure how to help with that"). The cause: the Triage Agent has zero Knowledge sources and zero Tools by design — its job is purely to classify and route based on its instructions. With ungrounded responses blocked, the LLM had no permitted response path, so every query fell through to Fallback.

This contradicted the Day 2 plan, which had loosely intended to flip ungrounded OFF for "safety." That intent was based on a single-agent mental model — the kind of agent that answers user questions from grounded sources. It does not apply to a router-style agent whose entire purpose is instruction-driven classification.

The setting is per-agent, not per-system. Different agents in the AgentOps Helpdesk system will need different values.
## Decision
"Allow ungrounded responses" is set per-agent based on the agent's architectural role:

- **Triage Agent** — ON. Routing decisions are produced by the LLM following its instructions. There is no grounding source to be "grounded in." Ungrounded responses are the only mechanism by which this agent can do its job.
- **Knowledge Agent** (Week 2-3) — OFF. The Knowledge Agent will be grounded in Azure AI Search (or Foundry RAG pipeline). Ungrounded responses must be blocked so the agent cannot hallucinate answers when retrieval fails.
- **ServiceNow Agent** (Week 2) — OFF. All responses derive from ServiceNow API tool calls. The agent must refuse when no tool result is available, not improvise.
- **Remediation Agent** (Week 3) — OFF. Actions are gated by tool calls and Power Automate flows. Ungrounded improvisation here is a safety risk.
- **Escalation Agent** (Week 3) — Decision deferred. Likely OFF, but depends on whether the agent generates handoff context from instructions only or grounds in conversation history + ticket data.
## Reasons
- The setting's correct value is a function of the agent's architecture, not a global safety toggle. Routers ground in instructions; specialists ground in sources or tools.
- For instruction-driven routers, OFF is structurally broken — it disables the only response mechanism the agent has.
- For grounded specialists, ON is a safety hole — it allows the model to hallucinate when retrieval or tools fail, exactly the failure mode RAG is supposed to prevent.
- The "ungrounded = unsafe" mental model is correct for grounded agents and inverted for routers. Treating it as a global setting causes one or the other to break.
## Alternatives considered
- Apply OFF globally for safety. Rejected: breaks router-style agents (proven by Triage 0/10 eval).
- Apply ON globally for flexibility. Rejected: removes the safety property that grounded agents need; allows hallucination in Knowledge / ServiceNow / Remediation paths.
- Use the setting as a uniform default and override per-agent only in edge cases. Rejected: the per-agent value is not an edge case, it is determined by architectural role. Treating it as an override invites future drift.
## Consequences
- Each new agent's spec must explicitly state the value of this setting and the reason. This becomes part of the per-agent ADR or instructions doc.
- Eval design must account for this. A router agent with grounding OFF will appear broken; a grounded agent with grounding ON will appear to work but may be hallucinating. Eval reviewers must know the agent's role to interpret results correctly.
- Day 2's loose intent to flip Triage's setting OFF is formally rejected. The correction is captured here so future Claude sessions and future-me do not repeat the mistake.
## When to revisit
- When a new agent is added — set per-agent value at creation time, document in the agent's spec.
- When an agent's architectural role changes (e.g., Triage gains Knowledge sources for some reason) — re-evaluate the setting.
- When Foundry custom evaluation harness is built (Week 3) — eval metrics must be role-aware so this setting's effect is interpreted correctly.
