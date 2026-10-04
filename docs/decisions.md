# Design decisions

Short record of the choices made, why, and what was rejected.

### D1: Separate lab tenant and developer environment
**Decision:** Build in a dedicated Microsoft 365 tenant with a Power Platform developer environment (`Portfolio-Dev`), never in an employer tenant or the default environment.
**Why:** Clean ownership, full admin control, and a proper Dev to Test ALM story. The default environment is shared and unsuited to solution-based development.

### D2: One shared Dev environment, one solution per project
**Decision:** Projects are separated by solution, not by environment. A single publisher (`ezm`) is used across solutions.
**Why:** Mirrors a real CoE shared-dev setup and stays within the developer-environment limit. Each solution is exported to Test to prove it carries no hidden cross-project dependencies.

### D3: Dataverse for data, SharePoint for content
**Decision:** Structured data (decisions, catalog, requests) in Dataverse; knowledge articles in SharePoint.
**Why:** Dataverse gives relationships, role-based security, field-level auditing and solution-aware ALM. SharePoint is the natural home for documents and a native Copilot Studio knowledge source.
**Rejected:** SharePoint lists for data: not solution components, weak relationships, view thresholds. (A list would be the right call for a low-governance tracker where premium licensing is the constraint.)

### D4: The AI proposes; rules, humans and least privilege decide
**Decision:** The agent only maps a request to a catalog fix. Eligibility is enforced by deterministic flow rules, business approval by the requester's manager, and execution by a service account scoped to the minimum needed.
**Why:** A prompt injection can at most cause a misclassification, which the rules, approver and scoped account then contain.

### D5: Remediation Catalog as an allow-list
**Decision:** Only fixes listed and Active in the catalog can run. The catalog stores risk level, approval requirement, execution channel, self-service flag and eligible departments.
**Why:** Policy is data, so a fix's behaviour changes without editing flows. New action codes require a solution change, which keeps new automated actions under change control.

### D6: API first, RPA only where there is no API
**Decision:** VPN access is granted by adding the user to an Entra ID security group through a cloud flow. Account unlock and MFA reset run as desktop flows against a legacy admin console (a stand-in app built for this project).
**Why:** Real identity operations belong in Entra / Microsoft Graph. RPA is reserved for systems without an API.

### D7: Risk-based approval
**Decision:** Low-risk fixes (account unlock) run without approval; medium and high-risk fixes require the requester's manager to approve.
**Why:** Approving everything defeats the purpose of automating the most common requests; approving nothing is unsafe.

### D8: Self-service only by default
**Decision:** Requester (from the Teams sign-in) and target user (from the message) are stored separately and must match when the catalog says Self Service Only.
**Why:** Blocks "do it for my colleague" requests and records any attempt in the audit trail.

### D9: Identity data stays in Entra ID
**Decision:** Departments, managers and group membership are read from Entra ID at runtime, not copied into Dataverse.
**Why:** A single source of truth; copies go stale. Only the automation's own policy (eligible departments) lives in Dataverse.

### D10: Every remediation creates a ServiceNow incident
**Decision:** The remediation flow creates an incident for every run and auto-resolves it on success.
**Why:** ServiceNow is IT's system of record. Needed for audit, provides a handover point if the fix fails, and makes automation ROI visible in IT's own reporting.
**Rejected:** Skipping incidents for low-risk fixes (could become a catalog-level policy later).

### D11: External systems referenced by key, not lookup
**Decision:** ServiceNow incident numbers and user emails are stored as text keys.
**Why:** Lookups only work within Dataverse. Syncing copies of external records would create stale duplicates. Virtual tables are the upgrade path if incidents ever need to behave like native rows.

### D12: Custom status choice instead of Status Reason
**Decision:** Remediation Request uses its own Request Status choice.
**Why:** Simpler to update from Power Automate. Trade-off: no built-in transition rules, so the flow enforces the order of states.

### D13: Ownership type per table
**Decision:** Agent Decision and Remediation Request are user/team owned; Remediation Catalog is organization owned.
**Why:** Transactional records belong to people and may need row-level security; the catalog is shared reference data.

### D14: All columns optional; validation in flows
**Decision:** No column is marked required.
**Why:** Rows are written by flows, and "required" is enforced only on forms. Real validation happens in the flows before writing.

### D15: Log the AI's request summary, not a raw message
**Decision:** Agent Decision stores an AI-generated Request Summary instead of the user's raw message.
**Why:** A conversation is multi-turn; any single message (e.g. "Outlook" in reply to a clarifying question) is meaningless on its own. The full verbatim conversation is already kept in Copilot Studio's conversation transcripts, linked by Conversation ID.

### D16: Deterministic values never come from the AI
**Decision:** When the agent calls a workflow, Requester Email and Conversation ID are bound to system variables; only judgement values (summary, category, reason) are generated by the AI. Category is constrained to a fixed dropdown.
**Why:** Identity and facts must not be open to model error or prompt injection.

### D17: Audit writes run on the automation's identity, not the user's
**Decision:** The logging tool uses maker-provided credentials, and the flow's Dataverse connection is embedded (the flow's own connection), not provided by the caller. In Test and Prod the connection reference maps to a least-privilege service account.
**Why:** Employees must not need, or get, write access to the audit tables. The requester is recorded as data (Requester Email); the write itself is done by the system. End-user credentials are reserved for actions that must respect the caller's own permissions.
**Accepted risk:** Copilot Studio flags maker-credential tools at publish time. The tool can only create a row in one table, and its identity inputs are bound to system variables, so the exposure is limited to that single insert.

### D18: Verify connection runtime mode in the solution before every deployment
**Decision:** Before any export to Test or Prod, the unpacked solution is checked: each agent flow's connection references must show `"runtimeSource": "embedded"` where D17 applies, and each agent tool must show `mode: Maker`.
**Why:** The credential mode is configured per consuming agent but materialised on the flow definition. Agent flows created in Copilot Studio were found to carry `invoker` even when the tool was set to maker credentials, and the designer screens did not show it. See [platform notes](platform-notes.md#1-agent-flow-runs-with-the-callers-connection-despite-maker-credentials).

### D19: One owner and one auth mode per shared flow
**Decision:** A flow reused by several agents has a single documented auth mode and a fixed input/output contract. Consumers that need a different mode get a thin wrapper flow calling a shared child flow.
**Why:** Two agents attached to the same flow with different credential modes overwrite each other's setting on the shared definition.

### D20: Naming
**Decision:** Agent flows are named `wf_<Agent>_<Action>` (e.g. `wf_Triage_LogRoutingDecision`), and the tool on the agent carries the same name. The plain-language tool description is what the orchestrator uses to choose the tool.
**Why:** One name across the agent, the flow and the solution makes components traceable; the description carries the meaning for the model.

### D21: Child agents for shared trust, a connected agent for Remediation
**Decision:** Knowledge and ServiceNow are child agents of the helpdesk agent. Remediation is a separate, connected agent.
**Why:** Child agents decompose one agent's job where the parts share trust and conversation context. Remediation changes identities, so it runs as its own agent with its own least-privilege credentials, approval rules, evaluation and release cycle; Triage can request a fix but never holds the permissions to make one.

### D22: Determinism in proportion to the cost of error
**Decision:** Hand-offs whose failure is harmless (Triage to Knowledge) are routed by the model within the conversation. Anything that changes an account runs through a Dataverse-triggered flow with eligibility rules and human approval.
**Why:** A misrouted how-to question costs one wrong answer and is still logged. A misrouted account change is a security incident. Asynchronous, rule-driven flows are auditable and cannot be talked into acting. A Dataverse trigger was rejected for the Knowledge hand-off because it runs outside the conversation, where the user is waiting for the answer.

### D23: Knowledge lives only in the specialist agent
**Decision:** The SharePoint knowledge source is attached to `ag_Knowledge` only, scoped to the *KB Articles* library; general knowledge and web search are off.
**Why:** If the routing agent holds knowledge it can answer directly and bypass classification and logging. Scoping to one library keeps retrieval precise and answers permission-trimmed to what each user can read.
