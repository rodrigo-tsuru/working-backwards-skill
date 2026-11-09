# ServiceNow SPM Strategic Alignment Check

Mechanics for Stage 3b of the working-backwards skill. The goal is to answer one
question with evidence: **is the selected use case already part of the customer's
strategy as recorded in ServiceNow SPM?**

Read-only by default. Never create, update, or transition a record in the
customer's instance without explicit confirmation in the conversation.

---

## 1. Resolve the connection

Try these in order and stop at the first that works.

### a. ServiceNow MCP server

If the session has ServiceNow MCP tools available (names usually contain
`servicenow` or `snow`), use them. They already hold the instance and credentials.

### b. Table API over `curl`

Expected environment variables:

| Variable | Meaning | Example |
|---|---|---|
| `SN_INSTANCE` | Instance host or subdomain | `acme.service-now.com` or `acme` |
| `SN_USER` / `SN_PASSWORD` | Basic auth credentials | read-only integration user |
| `SN_TOKEN` | OAuth bearer token, used instead of user/password | |

Check what exists before asking the customer for anything:

```bash
env | grep -iE '^SN_|servicenow' | sed 's/=.*/=<set>/'
```

Build a base URL and a reusable auth flag:

```bash
SN_HOST="${SN_INSTANCE%%.service-now.com*}.service-now.com"
SN_BASE="https://$SN_HOST/api/now"
# Basic auth
SN_AUTH=(-u "$SN_USER:$SN_PASSWORD")
# or OAuth
SN_AUTH=(-H "Authorization: Bearer $SN_TOKEN")
```

Validate the connection with a cheap call before running the real queries:

```bash
curl -sS --fail-with-body "${SN_AUTH[@]}" \
  "$SN_BASE/table/sys_user?sysparm_limit=1&sysparm_fields=sys_id"
```

Treat the result honestly: `401` means bad credentials, `403` means the
integration user lacks read ACLs on that table, `404` means the table or plugin
is not present on this instance. Report which one happened — do not retry blindly
and do not invent a result.

### c. Manual fallback

If no MCP server and no usable credentials, do not guess and do not skip the
check. Ask the customer:

"I cannot reach your ServiceNow instance from here. To run the strategic
alignment check I need either read access to your SPM tables, or the list of
your current strategic objectives and goals. Can you export or paste the active
strategic objectives, portfolio goals, and any related demands or ideas?"

Then run the matching in section 3 against what they provide, and record in the
final document that the check was done from a customer-supplied extract rather
than a live query, with the date of the extract.

---

## 2. Where the strategy lives

SPM spreads strategic intent across several tables, and which ones exist depends
on the instance's release and activated plugins. Discover first, then query:

```bash
curl -sS "${SN_AUTH[@]}" \
  "$SN_BASE/table/sys_db_object?sysparm_fields=name,label&sysparm_limit=200&sysparm_query=nameSTARTSWITHsn_gf^ORnameSTARTSWITHpm_^ORnameSTARTSWITHdmn_^ORnameSTARTSWITHim_^ORlabelLIKEstrateg^ORlabelLIKEobjective"
```

Commonly present tables, in the order worth checking:

| Table | Holds | Why it matters |
|---|---|---|
| `sn_gf_goal` | Goals (Goal Framework), including strategic goals and their hierarchy | The primary record of "what the organization committed to" |
| `sn_gf_metric` / `sn_gf_target` | Metrics and targets attached to goals | Lets you compare the North Star metrics to committed targets |
| `pm_portfolio` | Portfolios | Which portfolio would own this work |
| `pm_program` | Programs | Existing program the use case could land in |
| `pm_project` | Projects | Whether something overlapping is already funded or in flight |
| `dmn_demand` | Demands | Whether this idea was already submitted, approved, or rejected |
| `im_idea` | Ideas (Innovation Management) | Whether the idea exists upstream of a demand |

Anything the discovery query returns with a strategy-like label (for example a
Strategic Planning application's own objective or plan tables) should be queried
too. Table names differ between releases; trust the discovery output over this
list, and tell the customer which tables you actually read.

---

## 3. Run the check

Every search in this section follows `servicenow-search.md`: sample the data
first, keyword sets in each language the records use, every term on every
field, page to the end, and a name sweep before any negative verdict.

### 3a. Pull the active strategy

```bash
curl -sS "${SN_AUTH[@]}" \
  "$SN_BASE/table/sn_gf_goal?sysparm_query=active=true&sysparm_display_value=all&sysparm_limit=100&sysparm_offset=0&sysparm_fields=sys_id,number,name,short_description,parent,owner,state,start_date,end_date"
```

Repeat for each strategy-bearing table found in section 2, narrowing by
`active=true` or by the current fiscal window where the table has dates. Count
first and page with `sysparm_offset` until you have read every row. Check the
schema before filtering: a field such as `active`, `state` or `parent` may not
exist on that table, and a filter on a missing field is ignored without error.
Read the targets of every candidate goal too (for example `sn_gf_goal_target`),
because the target names often state the intent more precisely than the goal.

If a goal's `number` is empty, cite it by name and `sys_id`.

### 3b. Keyword search for the idea itself

Global search covers several tables in one call:

```bash
curl -sS "${SN_AUTH[@]}" \
  "$SN_BASE/globalsearch/search?sysparm_search=<2-4+keywords+from+the+use+case>&sysparm_limit=20"
```

If global search is unavailable on the instance, fall back to per-table
`sysparm_query` filters:

```bash
curl -sS "${SN_AUTH[@]}" \
  "$SN_BASE/table/dmn_demand?sysparm_limit=20&sysparm_display_value=all&sysparm_fields=number,short_description,state,strategic_alignment,portfolio&sysparm_query=short_descriptionLIKE<keyword>^ORdescriptionLIKE<keyword>"
```

Build the keyword sets as `servicenow-search.md` section 2 describes: the use
case, the customer's original request word for word, the tools from Stage 1,
and synonyms, in every language the records are written in. Each term goes on
every populated text field (`short_description`, `description`, `name`). One
vocabulary mismatch must not produce a false "not in strategy".

### 3c. Classify the result

| Verdict | Evidence required |
|---|---|
| **Aligned** | A specific active goal or objective record whose intent covers the use case, cited by number and name, plus how the North Star metric maps to that goal's metric or target |
| **Partially aligned** | A matching goal exists but the use case serves a different user, metric, or timeframe than the goal commits to — state the gap precisely |
| **Already in flight** | A demand, idea, project, or program covering the same scope already exists — cite its number and state, and say whether this session duplicates it |
| **Not aligned** | Every strategy record was read and none covers the use case. Say which tables, how many records, which keyword sets (and their languages) you used, and that the name sweep ran, so the negative is verifiable |

A verdict with no record number behind it is not a verdict. If the data does not
support a conclusion, say "inconclusive" and name what is missing.

---

## 4. Optional write-back

Only after the customer explicitly asks, and only when the use case is aligned or
the deviation was formally accepted. Show the exact payload and get a yes before
sending it.

```bash
curl -sS -X POST "${SN_AUTH[@]}" -H 'Content-Type: application/json' \
  "$SN_BASE/table/im_idea" \
  -d '{"short_description":"<use case name>","description":"<Today Statement + North Star>"}'
```

Confirm the created record number back to the customer. If the POST fails,
report the status and body as returned — do not describe the record as created.
