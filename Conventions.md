# Repo conventions

Conventions enforced across this repo. Follow these for any new files or content.

## Dates

Format: DD-MMM-YYYY (e.g., 07-May-2026).

Applies to:
- Filenames containing dates
- Headers and dates inside markdown files
- ADR dates
- Evaluation report dates
- Commit messages where dates are referenced

Why: avoids ambiguity (US vs EU date order), reads quickly, scannable in folder listings.

## File naming

- Lowercase
- Hyphens between words (kebab-case)
- Date suffixes use DD-MMM-YYYY format
- Examples: triage-baseline-evaluation.md, day-2-07-May-2026.md

## Folder structure

/agents/<agent-name>/      Agent definitions, instructions, tests
/docs/adr/                 Architecture Decision Records
/docs/evaluations/         Evaluation reports per agent per run
/docs/journal/             Daily build journal
/docs/architecture/        System diagrams, data flows
/docs/governance/          DLP policies, security model, RAI checks
/solutions/                Power Platform Solution exports (added Week 2)
/scripts/                  Setup, deployment, utility scripts
/.github/workflows/        GitHub Actions CI/CD (added Week 4)

## Commit messages

Style: imperative, present tense, lowercase first word.

Format: type: description

Types:
- add — new file or content
- update — modify existing
- fix — correct an issue
- docs — documentation only
- refactor — restructure without changing behavior
- chore — maintenance, renames, conventions

Examples:
- add: triage agent baseline evaluation
- fix: escalate routing in triage prompt
- chore: rename files to DD-MMM-YYYY format
