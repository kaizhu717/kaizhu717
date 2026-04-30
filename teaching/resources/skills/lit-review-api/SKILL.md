---
name: lit-review-api
description: Use when the user asks for a literature review, citation search, or annotated bibliography on a mainstream research topic well-covered by OpenAlex. Performs grounded retrieval via the OpenAlex API, verifies every DOI at Crossref, and drops unverified references rather than "fixing" them. Never returns citations from model memory.
allowed-tools: Bash
---

# Literature Review Skill (API-grounded, Path A)

## When to use

The user has asked for papers on a research question and wants a verified, citable bibliography --- not a conversational summary. Typical phrasings: "do a lit review on X", "find me recent papers on X", "build an annotated bibliography for my dissertation topic".

If the user instead hands you a draft bibliography and asks you to check it, use the companion skill `verify-citations` instead.

## Non-negotiable rules

1. Every returned citation must come from an OpenAlex API response, never from model memory.
2. Every returned citation must pass the Crossref DOI verification gate in Step 4.
3. If no results survive verification, report that honestly and stop. Never "fill in" with plausible-looking papers.

## Workflow

### Step 1. Clarify the query (at most one question)

If the user's question is vague (e.g., "papers on retail"), ask ONE clarifying question about scope: time window, discipline, or methodology. Then proceed. If the question is already specific, skip this step.

### Step 2. Query OpenAlex

URL-encode the search string and hit OpenAlex. OpenAlex is free, keyless, and covers ~250M scholarly works including the social sciences. Add a `mailto` parameter to stay in the polite pool.

```bash
QUERY="scarcity messaging online retail"
ENCODED=$(python3 -c "import urllib.parse,sys; print(urllib.parse.quote(sys.argv[1]))" "$QUERY")
curl -s "https://api.openalex.org/works?search=$ENCODED&per_page=25&mailto=student@example.com" > results.json
```

Parse `results.json` with Python (stdlib only). For each result, extract:

- `id` (OpenAlex work ID)
- `doi`
- `title`
- `authorships[].author.display_name` (first author surname for the verification gate)
- `publication_year`
- `host_venue.display_name` (journal/venue)
- `cited_by_count`
- `abstract_inverted_index` (decode to plain text for the TL;DR)

### Step 3. Triage

Score each result 0--3 on relevance to the user's question. Keep entries with score ≥ 2. Drop the rest. Report how many were retrieved and how many survived triage.

### Step 4. Verify DOIs at Crossref (the verification gate)

For every entry that passed triage, confirm the DOI resolves at Crossref:

```bash
curl -s -o /dev/null -w "%{http_code}" "https://api.crossref.org/works/$DOI"
```

- HTTP 200 → keep.
- Any other status (404, 500, timeout) → **drop the entry entirely.** Do not attempt to substitute, correct, or "fix" a broken DOI.

Track how many entries were dropped at this gate.

### Step 5. Write the annotated bibliography

Produce a markdown file with this structure:

```
# Literature Review: <query>

**Retrieved:** N  |  **Kept after triage:** M  |  **Verified at Crossref:** K

## Verified references

1. Author, A., & Author, B. (2023). Title of the paper.
   *Journal Name*, 12(3), 45--67. [https://doi.org/<doi>](https://doi.org/<doi>)
   TL;DR: one-sentence summary from the abstract.

2. ...

## Disclosure

- Search date: YYYY-MM-DD
- Retrieval source: OpenAlex API (https://api.openalex.org)
- Verification source: Crossref API (https://api.crossref.org)
- Model: <model name>
- Retrieved: N  |  Kept after triage: M  |  Dropped at verification: N - K
```

### Step 6. Never invent

If zero entries survive verification, the output is:

> No verified references found for this query. This does not mean no papers exist --- it means the OpenAlex search did not surface any whose DOI resolves at Crossref. Try: broadening the query, widening the year window, or searching a different database.

Never, under any circumstance, list papers from model memory. The entire value of this skill is that every reference is traceable to a live API response.

## Notes for extending

- **Year filter:** append `&filter=from_publication_date:2020-01-01` to the OpenAlex URL.
- **Snowballing one hop:** for the top 3 verified papers, fetch `https://api.openalex.org/works/<id>` and pull `referenced_works` for backward citations.
- **Marketing journal filter:** append `&filter=host_venue.issn:XXXX-XXXX` for a specific journal ISSN.
