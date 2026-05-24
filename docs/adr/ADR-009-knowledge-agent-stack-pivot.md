# ADR-009 — Knowledge Agent Stack Pivot (Foundry → Open Stack)

**Status:** Accepted
**Date:** 24-May-2026
**Supersedes:** —
**Superseded by:** —
**Related:** ADR-001 (LLM as primary orchestrator)

## Context
- Azure free credits (₹18,909) expire 25-May-2026 — unused
- Foundry was the planned stack for Knowledge Agent (RAG, eval harness, BYO model) per Week 3 plan
- Foundry usage post-credit-expiry = recurring cost (Azure AI Search, inference, workspace add-ons)
- Job market reality: LangChain / LlamaIndex / CrewAI / OpenAI-direct have significantly higher demand than Foundry-specific roles in India market
- Foundry deferred to Week 3 — no Day 1-11 work touches it, so swap cost is zero

## Decision
- **Drop Foundry from project scope**
- **Knowledge Agent built on open-stack RAG** — choice between LangChain and LlamaIndex deferred to Week 3 (research-driven)
- Eval harness: open tools (PromptFoo, DeepEval, or Python scripts) instead of Foundry custom harness
- CPS remains for multi-agent orchestration — Triage, ServiceNow, Remediation, Escalation unchanged

## Why
- **Zero invested cost** — Foundry not yet touched
- **Same architectural patterns** — RAG = embeddings + vector store + retrieval + LLM. Tool-agnostic skill
- **Better learning surface** — open-stack exposes chunking strategy, embedding model, retrieval logic explicitly; Foundry hides them
- **Higher employability** — LangChain/LlamaIndex on resume opens more freelance + full-time doors
- **No vendor lock-in narrative** — interview defensible: "tool selection driven by skill demonstrated, not stack loyalty"
- **Removes credit-expiry anxiety** — no recurring Azure spend post-trial

## Alternatives considered
- **Keep Foundry, pay post-expiry** — rejected. Recurring spend (~₹500-1500/month) for vendor-wrapped equivalent of free open-stack patterns
- **Switch entire project to LangChain** — rejected. CPS multi-agent + ServiceNow integration are the project's enterprise differentiator; throwing away 11 days of work for marginal gain
- **Use CPS native Knowledge tab for Knowledge Agent** — rejected. Black-box RAG, no eval surface, no portfolio value

## Trade-offs accepted
- AB-620 still primary cert (Microsoft-aligned) — but project shows hybrid stack, which is realistic for the market
- Knowledge Agent stack TBD until Week 3 — small risk of analysis paralysis; mitigated by 1-day cap on decision
- Foundry deprioritization removes one CV-worthy Microsoft-specific skill — replaced by broader-market open-stack skill (net positive)

## Consequences
- Project narrative refined: "enterprise multi-agent on Microsoft Copilot Studio + ServiceNow, with open-stack RAG for knowledge retrieval"
- Week 3 plan re-scoped — LangChain or LlamaIndex evaluation as Day 1 of Week 3
- Azure credits expire 25-May without strategic use — accepted loss
- Insight #34 candidate: tool selection driven by skill + market, not vendor narrative
- CrewAI standalone project (post-AB-620) remains in plan — now better aligned with main project's open-stack direction
