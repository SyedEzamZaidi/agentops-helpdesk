# Solution source

Unmanaged export of the **AgentOps Helpdesk** solution from Portfolio-Dev, unzipped so changes are reviewable as diffs. Version **1.0.0.7** (17 Oct 2026).

| Folder | Contains |
|---|---|
| `src/Workflows/` | Cloud and agent flows (`wf_*`) |
| `src/bots/`, `src/botcomponents/` | The agent, its topics, tools, child agent (`agent.Agent` = `ag_Knowledge`), knowledge source and evaluation test cases (`mspva_*`) |
| `src/customizations.xml` | Tables, choices, connection references |
| `src/solution.xml` | Solution metadata, publisher `ezm` |

Deployable builds (managed) are attached to GitHub releases, not committed here.

## Pre-deployment check (D18)

Verified on this export:
- every flow connection is `"runtimeSource": "embedded"` (Triage logging, UpdateDecision, Remediation submit and process);
- both agent tools are `mode: Maker`;
- `wf_Remediation_ProcessRequest` uses connection references for Dataverse, Office 365 Users, Approvals, Microsoft Entra ID and Teams, so Test and Prod map them to a service account at import;
- no credentials are included (connection references carry names only).

Topic schema names are fixed at creation, so *Request Account Fix* is stored as `topic.Untitled`; the display name is what the agent and instructions use.

To rebuild a zip: zip the contents of `src/` (not the folder itself) and import it as an unmanaged solution.
