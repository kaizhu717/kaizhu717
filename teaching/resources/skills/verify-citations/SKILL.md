---
name: verify-citations
description: Use when the user has a draft bibliography or list of citations from any source (web search, another LLM, a paper's reference section, a prior conversation, a file) and wants each entry verified against Crossref. Classifies every entry as VERIFIED, MISMATCH, or FABRICATED. Does NOT fix or substitute fabrications --- it flags them and leaves the decision to the user.
allowed-tools: Bash, Read
---

# Citation Verification Skill (Path B companion)

## When to use

A list of citations already exists and needs to be audited before it can be trusted. Typical scenarios:

- Claude Code just used web search to answer a lit-review question and produced a draft bibliography.
- The user pasted a reference list from a paper and asked "are these real?".
- Another LLM was asked for sources and the user is suspicious.
- An old bibliography needs to be re-checked for broken DOIs.

If the user has not yet retrieved any citations and wants a fresh lit review, use the companion skill `lit-review-api` instead.

## Non-negotiable rules

1. Every entry must be classified into exactly one of: VERIFIED, MISMATCH, FABRICATED.
2. Never "fix" a FABRICATED entry by substituting a similar-looking real paper. Flag it and stop.
3. Never downgrade a MISMATCH to VERIFIED because "it is close enough". Record what was wrong.

## Workflow

### Step 1. Parse the input

Accept any common format: APA, BibTeX, Chicago, plain text, or a markdown list. For each entry, extract:

- First author surname
- Year
- Title
- Venue (journal/conference name)
- DOI (if present)

If a field is missing, record it as `null` and proceed --- missing fields are informative.

### Step 2. Resolve DOIs at Crossref

For entries that have a DOI:

```bash
curl -s "https://api.crossref.org/works/$DOI" > crossref.json
```

Parse the JSON and check:

- First author surname: exact match (case-insensitive) against Crossref's `author[0].family`.
- Year: exact match against `issued.date-parts[0][0]`.
- Title: ≥ 80% fuzzy match against `title[0]` (use `difflib.SequenceMatcher` from Python stdlib).

Record which fields passed and which failed.

### Step 3. Fall back to title search for entries without a DOI

For entries with no DOI (or where the DOI returned HTTP 404), search Crossref by title:

```bash
ENCODED_TITLE=$(python3 -c "import urllib.parse,sys; print(urllib.parse.quote(sys.argv[1]))" "$TITLE")
curl -s "https://api.crossref.org/works?query.title=$ENCODED_TITLE&rows=3" > fallback.json
```

For each of the top 3 hits, compare first author and year against the claimed citation. If any hit matches both, record its DOI as evidence.

### Step 4. Classify every entry

Apply these rules in order:

- **VERIFIED** --- DOI resolved at Crossref AND first author, year, and title (≥80%) all match. Or title-search found a hit where author + year match.
- **MISMATCH** --- The paper exists (DOI resolved or title search found a confident match) BUT at least one claimed field is wrong. Record exactly which fields differ and what Crossref says.
- **FABRICATED** --- No DOI resolution AND no title-search match. The paper cannot be located in Crossref.

A MISMATCH is not a fabrication. The paper is real --- the citation just has errors. These are often typos or misremembered years, and the user may want to fix them.

### Step 5. Report

Produce a markdown file with a summary header and a table:

```
# Citation Verification Report

**Input:** <source file or description>  |  **Entries audited:** N
**Verified:** V  |  **Mismatch:** M  |  **Fabricated:** F

| # | Original citation | Verdict | Evidence |
|---|---|---|---|
| 1 | Author, A. (2023). Title. *Venue*. | VERIFIED | https://doi.org/... |
| 2 | Author, B. (2022). Title. *Venue*. | MISMATCH | year should be 2021; https://doi.org/... |
| 3 | Author, C. (2024). Fake Title. *Venue*. | FABRICATED | no Crossref match for title or author+year |
| ... |

## Recommended actions

- **Verified (V):** safe to use as-is.
- **Mismatch (M):** real papers, but fix the flagged fields before citing.
- **Fabricated (F):** drop from the bibliography. Do not cite.
```

### Step 6. Never substitute

If an entry is FABRICATED, the output is "FABRICATED --- not found in Crossref". Do not suggest "a similar paper" or "did you mean...". The user decides what to do. The job of this skill is to report, not to repair.

## Notes for extending

- **Semantic Scholar fallback:** for papers that Crossref cannot find but might be on arXiv or pre-print servers, add a secondary lookup at `https://api.semanticscholar.org/graph/v1/paper/search?query=<title>`.
- **Author-list verification:** strengthen the check by comparing all authors, not just the first surname.
- **Export:** after verification, emit a cleaned BibTeX file containing only VERIFIED entries (with any MISMATCH corrections merged in, if the user chooses).
