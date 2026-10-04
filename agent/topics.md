# Topics

Under generative orchestration the planner selects topics by their description, so each description is written as a routing contract with explicit exclusions.

## Escalate (system topic)

**Trigger:** On Talk to Representative

**Description:**
```text
Use only when the user explicitly asks to speak to a human, a live agent or a person (for example "talk to a person", "connect me to an agent"). Do not use for reporting problems, security incidents, hacked accounts, phishing, outages or any IT request; those must be classified and logged instead.
```

**Why scoped:** with the default description ("Talk to agent, Talk to a person, …") the planner sent security incidents here, which bypassed the logging tool and left no Agent Decision row ([20-case run](../eval/results/2026-10-04-triage-standard-20case.md)). After scoping, all Escalate cases are logged ([rerun](../eval/results/2026-10-04-triage-standard-20case.md#rerun-after-fixes)).

**Next:** replace the placeholder message with a hand-off that logs the request first and returns the IT service desk contact.
