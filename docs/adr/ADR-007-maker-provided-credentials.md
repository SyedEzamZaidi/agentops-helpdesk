# ADR-007 — Maker-Provided Credentials for ServiceNow Agent Tool Connections

**Status:** Accepted
**Date:** 19-May-2026
**Supersedes:** —
**Superseded by:** —
**Related:** ADR-005 (Client Credentials production target), ADR-006 (Authorization Code tactical deferral)

## Context
- Day 10 wiring of custom connector into ServiceNow Agent surfaced a third credential option not previously evaluated
- ADR-005 specified Client Credentials grant + service account `agentops_servicenow_svc` as the production auth target
- ADR-006 tactically deferred to Authorization Code grant with admin user due to ServiceNow Zurich Entity Profile UX trap
- Copilot Studio tool configuration exposes a "Credentials to use" setting with two choices: **End user credentials** vs **Maker-provided credentials**
- This is a Power Platform connection-binding layer, distinct from the OAuth grant type at the ServiceNow layer

## Decision
- Use **Maker-provided credentials** on all 4 ServiceNow Agent tools (Create, Get list, Get by ID, Update)
- The maker's connection (currently bound to admin user via Authorization Code grant per ADR-006) is reused for every agent invocation
- All API calls to ServiceNow authenticate as the same admin identity regardless of which end user is chatting

## Why
- **End user credentials would force per-user OAuth dance** — every end user has to log into ServiceNow before any tool fires; breaks at multi-user scale (Insight #12 from project context)
- **Maker-provided is the closest approximation to Client Credentials achievable without resolving the Zurich Entity Profile blocker** — one identity for all calls, no per-user setup
- **Architecturally consistent with trusted-subsystem pattern** — service account (or maker proxy) has broad permissions, agent enforces per-user filtering at application layer
- **No code/architecture change required to swap to Client Credentials later** — only the underlying connection's auth config changes; tool wiring stays identical

## Trade-offs accepted
- Connection identity is tied to a human admin account, not a dedicated service account — if admin password rotates, MFA changes, or account is disabled, all 4 tools break
- Audit trail at ServiceNow's `sys_updated_by` field shows `admin` for every modification, not a dedicated service account — origin attribution recovered through `u_agent_name` custom field at application layer (already in design)
- Token refresh depends on admin user's session validity rather than a stable service-account credential

## Identity layering (locked terminology)
Four distinct identity layers exist per ticket. Maker-provided solves one of them:

| Layer | Purpose | Mechanism |
|---|---|---|
| Connection identity | Auth to ServiceNow | Maker-provided (admin's OAuth token via Authorization Code) |
| Business identity | Whose ticket this is | `caller_id` field — resolution pending ADR-008 |
| Operating identity | Who modified the record | `opened_by` / `sys_updated_by` — set by ServiceNow automatically |
| Origin identity | Which agent originated the action | `u_agent_name` custom field — set at application layer |

## Production path
- Client Credentials grant with `agentops_servicenow_svc` (ADR-005) remains the production target
- Swap is config-only: rebind the Power Platform connection from Authorization Code to Client Credentials grant after the Zurich Entity Profile binding is resolved
- All 4 tool wirings, agent instructions, audit pattern, and three-layer authorization model remain unchanged
- Maker-provided + Authorization Code = stepping stone. Maker-provided + Client Credentials = production end state.

## Alternatives considered
- **End user credentials** — rejected: multi-user OAuth friction makes it operationally broken at scale
- **Custom auth-handling Power Automate flow** — rejected: adds moving parts, breaks LLM-as-orchestrator principle (ADR-001)
- **Wait until Zurich Entity Profile binding is scripted** — rejected: blocks Day 10 ship; tactical deferral pattern (ADR-006) is already in play and adequate

## Consequences
- Day 10 ships with maker-provided + Authorization Code, fully functional for single-maker testing
- Production-grade auth requires ADR-005 Client Credentials swap before broader rollout
- Solution-based ALM (Week 4) must include connection rebinding step for environment promotion
- Per-environment connections required (dev/test/prod each bound to environment-specific maker or service account)
