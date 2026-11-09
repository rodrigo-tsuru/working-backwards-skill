# PRF Template (Problem, Requirements, Forward Plan)

Use this template when the customer chose Option 2.

```markdown
# Intent: [Project or Feature Name]

**Author:** [customer name]    **Date:** [today]    **Status:** Draft

**Type:** Greenfield | Brownfield (from Stage 3c)    **Target release:** [from North Star timeline]

---

## P - Problem

### Context

[One paragraph from Stage 1: the current situation, what exists today, why this matters now]

### Strategic alignment (ServiceNow SPM)

**Verdict:** Aligned | Partially aligned | Already in flight | Accepted deviation

| Strategy record | Name | How this use case serves it |
|---|---|---|
| [GOAL/number from Stage 3b] | [record name] | [which part of the objective this moves] |

- **Checked on:** [date]  **Source:** live SPM query | customer extract dated [date]
- **Tables read:** [e.g. sn_gf_goal, dmn_demand, pm_project - and how many active records]
- **North Star to goal metric:** [how the Stage 5 metric maps to the goal's own target]
- **Existing related records:** [demand/idea/project number and state, or "none found"]
- **Accepted deviation (only if verdict is Accepted deviation):** [name, role] accepted on [date] because [reason, in their words]

### Application portfolio (ServiceNow APM)

**Verdict:** New application | Extends existing | Replaces existing | Accepted overlap

| Business application | Name | Lifecycle stage | Owner | Relation to this use case |
|---|---|---|---|---|
| [APM/number from Stage 3c] | [name] | [stage] | [business / IT owner] | [extended / replaced / overlaps / integration point] |

- **Checked on:** [date]  **Source:** live portfolio query | customer extract dated [date]
- **Capability:** [business capability served, and how many active apps already provide it]
- **Scope read:** [tables, keyword sets, number of records read - required for New application]
- **Accepted overlap (only if verdict is Accepted overlap):** [name, role] accepted on [date] because [reason, in their words]

### Problem statement

[Expanded Today Statement from Stage 2. One or two paragraphs, stated from the user perspective, not the solution perspective]

### Target users

| User type | What they need | Pain today |
|---|---|---|
| [from Stage 1] | [from Stage 1 goals] | [from Stage 1 pain points] |
| [from Stage 1] | [from Stage 1 goals] | [from Stage 1 pain points] |

### Success metrics

| Metric | Target | Measured how |
|---|---|---|
| [from Stage 5] | [target] | [measurement method] |
| [from Stage 5] | [target] | [measurement method] |
| [from Stage 5] | [target] | [measurement method] |

---

## R - Requirements

### In scope (MVP)

- [Feature 1, derived from Stage 3 solution direction and Stage 4 future experience]
- [Feature 2]
- [Feature 3]

### Out of scope (explicit)

| Excluded feature | Reason | Target phase |
|---|---|---|
| [feature] | [why out now] | [Phase 2/3] |

### Constraints

- **Must use:** [languages, frameworks, services - ask customer if not already stated]
- **Must not change (brownfield only):** [existing APIs, schemas, services of the application from Stage 3c that must stay untouched]
- **Compliance / security:** [e.g. PCI, LGPD, SOC2, internal security baseline]
- **Non-functional targets:** [availability, latency, throughput, data residency]

### Assumptions

- [Assumption 1, state explicitly so AI-DLC can challenge it]
- [Assumption 2]

---

## F - Forward plan

### Open questions (pre-declared ambiguities)

- [Question 1, anything that came up during the session that remains undecided]
- [Question 2]

### Risks

| Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|
| [risk] | H/M/L | H/M/L | [how to address] |

### Roll-out sketch

- **Phase 1 (MVP):** [what ships first]
- **Phase 2:** [next increment]
- **Phase 3+:** [aspirational]

### Stakeholders and approvals

| Role | Name | Approval needed on |
|---|---|---|
| Sponsor | [name] | Problem and MVP scope |
| SPM objective owner | [name] | Strategic alignment (or the accepted deviation) |
| Application owner (brownfield only) | [name, from Stage 3c] | Changes to the existing application |
| Product owner | [name] | Requirements and stories |
| Tech lead | [name] | Application and NFR design |
| Security | [name] | Extension opt-ins, gates |
| QA lead | [name] | Build and test instructions |
```
