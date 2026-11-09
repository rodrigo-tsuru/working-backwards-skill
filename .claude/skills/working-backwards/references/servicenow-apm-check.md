# ServiceNow Application Portfolio Check

Mechanics for Stage 3c of the working-backwards skill. The goal is to answer one
question with evidence: **does an application that already does this, or should
do this, exist in the customer's application portfolio in ServiceNow?**

Building a new application when the portfolio already holds one that covers the
same capability creates a redundant application that someone later has to
rationalize. Finding it now changes the project type (greenfield vs brownfield),
the scope, and the people who must approve.

Read-only by default. Never create, update, or retire a record in the customer's
instance without explicit confirmation in the conversation.

**Connection:** reuse the one resolved for Stage 3b (see
`servicenow-spm-check.md`, section 1: MCP tools first, then the Table API over
`curl` with `SN_BASE` / `SN_AUTH`, then a manual extract). Do not ask the
customer for credentials a second time.

---

## 1. Where the portfolio lives (CSDM)

| Table | CSDM class | Holds | Why it matters |
|---|---|---|---|
| `cmdb_ci_business_app` | Business Application | The logical application portfolio (APM) | The primary record of "which applications exist" |
| `cmdb_ci_business_capability` | Business Capability | Capability map | Lets you find apps by what they do, not by name |
| `cmdb_rel_ci` | Relationships | `Provided by` (capability to app), `Uses` (app to service instance) | Links capabilities, apps and deployments |
| `cmdb_ci_service_auto` (and children such as `cmdb_ci_service_discovered`) | Service Instance (Application Service) | Deployed, running stacks of an app | Shows whether the app is live and in which environments |
| `cmdb_ci_sdlc_component` | SDLC Component | Code repos and build units | Shows who maintains the code and where it lives |
| Digital Product tables (CSDM 5, plugin-dependent) | Digital Product | Product portfolio | Check if the discovery query returns them |

Discover what this instance actually has before querying:

```bash
curl -sS "${SN_AUTH[@]}" \
  "$SN_BASE/table/sys_db_object?sysparm_fields=name,label&sysparm_limit=100&sysparm_query=nameINcmdb_ci_business_app,cmdb_ci_business_capability,cmdb_ci_service_auto,cmdb_ci_sdlc_component^ORlabelLIKEdigital product^ORlabelLIKEbusiness application"
```

Field names vary between releases (for example `life_cycle_stage` /
`life_cycle_stage_status` in CSDM 4 and later, `install_status` /
`operational_status` in older data). Read the schema of `cmdb_ci_business_app`
once and use the fields that exist:

```bash
curl -sS "${SN_AUTH[@]}" \
  "$SN_BASE/table/sys_dictionary?sysparm_fields=element,column_label&sysparm_limit=200&sysparm_query=name=cmdb_ci_business_app"
```

With the ServiceNow MCP server, use `get_table_schema` for the same thing and
`list_ci` / `get_ci_relationships` for the queries below.

---

## 2. Run the check

Every search in this section follows `servicenow-search.md`. In short:

- Sample the portfolio first, to learn its size, language(s) and naming
  convention.
- Build keyword sets in **every language the records use**, from short stems
  (`capacit`, `treina`, `educa`, not `training` alone).
- Include the customer's **original request word for word**. The solution they
  first asked for is the one most likely to exist already.
- Run every term against `name` and every populated description field.
- Page to the end.
- Sweep all application names before any **New application** verdict, and
  before settling on a single candidate.

### 2a. Search by name and description

Derive the keyword sets from the use case name, the Today Statement, the
customer's original request, the user's workflow from Stage 1 (the tools they
named), and the capability the use case delivers. Application names are often
acronyms or internal brands that carry none of these words, so the description
search and the name sweep in 2e are what find them.

```bash
curl -sS "${SN_AUTH[@]}" \
  "$SN_BASE/table/cmdb_ci_business_app?sysparm_limit=50&sysparm_display_value=all&sysparm_fields=sys_id,number,name,short_description,owned_by,it_application_owner,business_criticality,life_cycle_stage,life_cycle_stage_status,install_status,user_base,application_type,platform&sysparm_query=nameLIKE<keyword>^ORshort_descriptionLIKE<keyword>^ORdescriptionLIKE<keyword>"
```

### 2b. Check the tools the users already work in

Every system the customer named in Stage 1 (question 4, "what tools do they use
at each step") is a candidate. Look each one up by name. If the use case changes
how users work in one of them, that app is in scope even if it does not match
the keywords.

### 2c. Search by capability

Find the business capability the use case serves, then the applications that
provide it:

```bash
# Capability
curl -sS "${SN_AUTH[@]}" \
  "$SN_BASE/table/cmdb_ci_business_capability?sysparm_limit=20&sysparm_fields=sys_id,name,short_description,parent&sysparm_query=nameLIKE<keyword>^ORshort_descriptionLIKE<keyword>"

# Applications that provide it ("Provided by" relationship, capability is parent)
curl -sS "${SN_AUTH[@]}" \
  "$SN_BASE/table/cmdb_rel_ci?sysparm_limit=50&sysparm_display_value=true&sysparm_fields=parent,child,type&sysparm_query=parent=<capability_sys_id>^child.sys_class_name=cmdb_ci_business_app"
```

Two or more active apps on the same capability is already a rationalization
signal. Report it even if neither one matches the use case exactly.

### 2d. For each candidate, read its state

For every application that plausibly overlaps, collect:

- **Lifecycle:** stage and status (Ideation, Design, Build, Deploy, Operational,
  End of Life). An app in End of Life, or being retired, changes the conversation.
- **Ownership:** business owner and IT application owner. They become
  stakeholders.
- **Deployments:** service instances it `Uses` (via `cmdb_rel_ci` with the app as
  parent), to tell whether it is actually running and in which environments.
- **Criticality and user base:** whether it is a large system that changes slowly
  or a small tool that is easy to extend.
- **Related work:** open demands or projects that reference it (`dmn_demand`,
  `pm_project` filtered on the app or its name), tying back to Stage 3b.

### 2e. Sweep the names

Keyword search finds only the words you thought of. List every application
name and number (`fields=number,name`, paged to the end; about 1,600 names fit
in two pages) and read the list for anything that could cover the capability,
the users, or the customer's original request. Read the description of each
candidate you pick out, and add any new vocabulary to 2a and run it again.
For portfolios over about 5,000 apps, sweep by `category`, `subcategory` or
`business_unit` instead (`servicenow-search.md` section 5).

### 2f. Classify the result

| Verdict | Evidence required |
|---|---|
| **New application** | Every search in 2a-2e was run, including the name sweep, and no active application covers the capability, the workflow, or the customer's original request. Say which tables, which keyword sets and languages, and how many records and names you read, so the negative can be checked |
| **Extends existing** | A specific application, cited by number and name, already serves these users or this workflow, and the use case adds capability to it. The project is brownfield on that app |
| **Replaces existing** | An application covers the scope but is end-of-life, being retired, or not fit for purpose. The use case is its successor. Retirement or migration of the old app becomes in scope or an explicit open question |
| **Duplicate** | An active application already does what the use case proposes. Building it would add a redundant app |
| **Inconclusive** | The portfolio could not be read, or it is too incomplete to support a negative (for example, the capability map is empty and app descriptions are blank). Name what is missing |

A verdict with no record number behind it is not a verdict. A thin or empty
portfolio is not proof that nothing exists: if `cmdb_ci_business_app` has very
few records, or most have no description, say so and ask the customer before
concluding **New application**.

---

## 3. Manual fallback

If Stage 3b fell back to a customer extract, ask for the portfolio too:

"I also need to check whether an application that already does this exists in
your portfolio. Can you export your business applications (name, description,
owner, lifecycle stage) from ServiceNow APM, or from wherever your application
inventory lives, filtered to the capability area we are working on?"

Run section 2f against what they provide, and record in the final document that
the check was done from a customer-supplied extract, with its date.

---

## 4. Optional write-back

Only after the customer explicitly asks, and only when the verdict is **New
application** and Stage 3b closed. Show the exact payload and get a yes before
sending it. Create the record in the Ideation stage. It is a planned application,
not a deployed one.

```bash
curl -sS -X POST "${SN_AUTH[@]}" -H 'Content-Type: application/json' \
  "$SN_BASE/table/cmdb_ci_business_app" \
  -d '{"name":"<use case name>","short_description":"<Today Statement>","life_cycle_stage":"<sys_id of the Ideation stage>"}'
```

On most instances `life_cycle_stage` is a reference field. Look up the sys_id
of the Ideation stage first (`life_cycle_stage` table, or the field's choice
list on older releases), or leave the field out and tell the customer it still
needs to be set.

Confirm the created record number back to the customer. If the POST fails,
report the status and body as returned. Do not describe the record as created.
