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

## Application portfolio gate (ServiceNow APM / CSDM)

Right after the strategy check, the skill checks whether an application that
already covers the use case exists in the customer's application portfolio
(Stage 3c). It searches Business Applications (`cmdb_ci_business_app`) by
keyword, looks up the tools the users named during the session, and searches by
Business Capability and the applications that provide it. The verdict (New
application, Extends existing, Replaces existing, Duplicate) sets the project
type to greenfield or brownfield and adds the application owner to the
approvers. A duplicate stops the session until the use case is reframed as an
extension or replacement, shown to be different, or a named stakeholder accepts
the overlap. It uses the same connection and the same read-only rules as the
strategy check.

# tech-env skill

Use this skill to document your technical environment and enhance the Construction phase accuracy. This skill speeds up reverse engineering
