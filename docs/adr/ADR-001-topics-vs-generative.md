# ADR-001: Topics vs Generative Orchestration

- Status: Accepted
- Date: 07-May-2026
- Author: Syed Ezam Zaidi

## Context

Copilot Studio agents have two response paths:

1. Topics — deterministic flows triggered by phrase matches. A topic owns the turn it fires on. Topics can also invoke Power Automate flows, set variables, and chain into other topics.

2. Generative AI — LLM-driven responses governed by the instructions field. Used when no topic matches.

Microsoft auto-creates several System Topics on new agents (Greeting, Escalate, Goodbye, Conversation Start, etc.) without explicit consent. These can silently override prompt-driven behavior, as discovered when a Triage Agent eval failure was traced to the auto-created Escalate topic intercepting LLM-bound classifications.

The key architectural question: when should a behavior live in a topic vs in the prompt?

## Decision

For the AgentOps Helpdesk project:

- The LLM (driven by instructions) is the primary orchestrator.
- Topics exist when ONE of the following applies:
  - The behavior must be deterministic (compliance, regulatory, audit-required)
  - The behavior triggers a Power Automate flow (genuine workflow handoff)
  - The behavior must call out to another agent or external system reliably
  - The trigger phrase set covers a common, well-defined business flow
- Auto-created System Topics are reviewed at agent creation and either edited, disabled, or deleted. None remain by accident.
- When both a topic and prompt could handle a request, the topic's response and the prompt's intended response must be aligned. Inconsistency between them is treated as a defect.

## Reasons

- Predictability over invisibility. Topics that exist by accident are silent overrides — these break trust in the system. Topics that exist on purpose are visible architectural choices.
- Topics earn their place when they invoke flows. A topic that just emits a hardcoded message is mostly redundant with the prompt; a topic that fires a workflow is genuinely doing different work.
- Modern Copilot Studio supports Generative Orchestration where the LLM treats topics as callable tools, choosing them based on context. This is the design we lean toward.
- Compliance and high-stakes domains require deterministic responses that prompt iteration cannot reliably guarantee. Topics serve this need.

## Alternatives considered

- Disable all topics, run pure LLM. Rejected: loses the legitimate use case of topics-as-flow-triggers.
- Topic-first design with LLM as fallback (legacy Power Virtual Agents pattern). Rejected: too rigid for the multi-agent classification system; topic explosion would become unmaintainable.
- Leave Microsoft's auto-created topics alone. Rejected: silent overrides have already broken eval results once.

## Consequences

- Higher LLM dependency for routine classification. Failures addressed by prompt iteration first, topic surgery only when prompt cannot solve it.
- Topics that DO exist must be intentional. Each topic gets a justification documented in its own ADR or referenced in the agent's spec.
- When adding multi-agent handoffs in Week 2, topics will become important again — they will be the trigger surface for Power Automate flows that route to sub-agents or external systems.

## When to revisit

- When adding compliance-required behaviors (mandatory disclaimers, mandatory escalation paths). Those become topics.
- When prompt iteration cannot reliably solve a classification edge case. The fallback is a topic for that specific case.
- When the multi-agent architecture matures (Week 2 onward) and Power Automate flow triggers proliferate.
