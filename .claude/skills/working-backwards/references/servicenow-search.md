# Searching ServiceNow without missing records

Shared rules for the searches in Stage 3b (`servicenow-spm-check.md`) and
Stage 3c (`servicenow-apm-check.md`). Both gates can return "nothing found",
and a wrong "nothing found" is worse than no check at all: it lets a duplicate
through with a verdict that looks like evidence.

The rules below come from real misses. Example: a search for training
applications used English stems (`educat`, `training`, `learning`) against a
portfolio written in Portuguese, and missed `VEDU - VIVO EDUCACIONAL`, whose
description reads "educação corporativa voltada à capacitação de
colaboradores". Every one of these rules would have caught it.

---

## 1. Read the data before you search it

Before building any keyword list, sample the table:

- `aggregate_records` (or `sysparm_count`) for the total, so you know how much
  you are searching and whether you need to page.
- 10-20 rows with `name` and the description field, to learn:
  - **The language or languages** the records are written in. Portfolios are
    often mixed: vendor product names in English, descriptions in the local
    language.
  - **The naming convention.** For example `ACRONYM - FULL NAME`, all caps, or
    internal brand names ("VIVO IA", "VEDU") that say nothing about what the
    app does.
  - **Which description fields are filled in** (`short_description`,
    `description`, `comments`). Search the ones that are actually populated.

## 2. Build keyword sets in every language the data uses

- Make **at least one full keyword set in each language** you found in step 1,
  not just a translation of one or two terms. If the session runs in Portuguese
  and the records are in Portuguese, the Portuguese set is the main one, not a
  fallback.
- Use **short stems**, because `LIKE` matches substrings: `capacit`, `treina`,
  `educa`, `aprend`, `conhec`, `onboard`, `trilha`. Do not use full words. For
  example, `educat` does not match `educação`.
- **Diacritics:** `LIKE` usually ignores case but not accents. Pick a stem that
  stops before the accent (`educa`, not `educação`), or include both forms
  (`código` and `codigo`).
- Cover **four sources of vocabulary**, one set each:
  1. The selected use case (name, Today Statement, North Star metric).
  2. **The customer's original request, word for word.** The solution they
     first asked for is often the one that already exists, even if the session
     reframed it.
  3. The tools and systems the users named in Stage 1.
  4. Synonyms and neighbouring concepts (for a training app: LMS, e-learning,
     academy, university, trilha, curso, certificação, educação corporativa).

## 3. Search every field with every term

Do not split terms between fields (for example some terms on `name` only and
others on `short_description` only). Each stem runs against `name` **and**
every populated description field:

```
nameLIKE<a>^ORshort_descriptionLIKE<a>^ORnameLIKE<b>^ORshort_descriptionLIKE<b>...
```

Build the encoded query from the full list mechanically, so no term is left out
of any field. If the query gets too long, split it by keyword set, not by field.

## 4. Page until the end

`limit` cuts results without warning. When a result returns exactly `limit`
rows, or the total from step 1 is larger than what you read, page with `offset`
until a page returns fewer rows than the limit. Never report "read all N
records" from a single page.

## 5. Sweep the names before concluding a negative

Keyword search only finds the words you thought of. Before any negative verdict
(**New application**, **Not aligned**, "no duplicates"), and before closing a
gate on one candidate when others may exist:

- If the table has up to about 5,000 rows, list **every name** (`fields=name,number`,
  limit 1000, paged) and read the list yourself, looking for anything that
  could plausibly cover the scope. Short or branded names ("VEDU", "MIA",
  "LBIA") only show up this way. Then read the description of every candidate
  you pick out.
- Above that size, group by `category`, `subcategory`, `business_unit` or
  `department` where they are populated, and sweep the names in the groups
  that are relevant.

Report that the sweep was done, and how many names it covered.

## 6. Treat a miss as a defect in the search

If the customer, or anyone else, names a record your search did not return:

1. Fetch it and say plainly that the search missed it and why (which language,
   field or term).
2. Add its vocabulary to the keyword sets and **run the whole search again**,
   because the same gap usually hides more than one record.
3. Re-run the classification on everything you now have. Do not just add the
   one record to the previous verdict.

## 7. Report what you searched

The "scope read" in each verdict lists the tables, the total number of rows, the
keyword sets **with their language**, the fields searched, the pages read, and
whether the name sweep ran. Someone reading the document should be able to
reproduce the search and see why a negative holds.
