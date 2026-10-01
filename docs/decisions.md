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
