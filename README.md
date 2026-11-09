# working-backwards-skill
Use this skill to define a use case for an AI-DLC workflow. Output: a markdown file with a structured problem to be solved.

## Strategic alignment gate (ServiceNow SPM)

The `working-backwards` skill checks the selected use case against the customer's
strategy in ServiceNow SPM (Stage 3b) before designing the future state. The
session does not advance past that stage until the use case maps to a strategy
record, is re-scoped to one that exists, or a named stakeholder formally accepts
the deviation — which is then recorded in the output document.

The check is read-only. To let it query live, set `SN_INSTANCE` plus either
`SN_USER`/`SN_PASSWORD` or `SN_TOKEN` (a read-only integration user is enough).
A ServiceNow MCP server, if available, is used in preference to the REST API.
Without either, the skill asks the customer for an extract of their active
strategic objectives and records that the check was done from that extract.

# tech-env skill

Use this skill to document your technical environment and enhance the Construction phase accuracy. This skill speeds up reverse engineering
