---
name: google-scholar-scout
description: Lightweight Google Scholar search runner that can operate standalone (topic-only) or with an optional query pack. Uses browser_subagent to execute a small set of Scholar queries and returns deduped paper results with links.
---

# Google Scholar Scout

## Overview

Run a **fast, structured Google Scholar search pass** to surface candidate papers for a new research topic.

This skill is the “execution” counterpart to an Academic Scout:
- If a **query pack** (keyword/query combos) is provided, it uses it.
- If **no query pack** is provided, it generates a minimal Scholar query set from the topic and proceeds.

The outputs are designed to be **actionable for the next step** (reading, Zotero import, building a literature map), not to be a full systematic review.

## Primary Goal

Given a topic (and optionally a query pack), produce:

- A **deduplicated list of papers** (title + Scholar link + best available full-text link/PDF link if present)
- Per-paper **metadata when visible** (authors, year, venue, “Cited by” count)
- A **query-to-paper trace** (which query surfaced which items)
- A shortlist of **high-priority candidates** for deeper reading

## Hard Constraints (Mandatory)

- You MUST call `browser_subagent` for any web action.
- You must do all searching only inside **Google Scholar**:
  - `https://scholar.google.com/`
- Do NOT use the external Google search bar or any other search engine page.
- Keep interactions minimal:
  - Use the Scholar search box and results pages only.
  - Avoid deep browsing across many tabs.
- Single-tab default:
  - Stay in one tab unless a result link is broken or you must open a PDF link.
- No fabrication:
  - Never invent titles, authors, years, venues, citation counts, or URLs.
  - If a field is not visible, output `"Unknown"`.
- Respect runtime limits:
  - Query budget and results-per-query limits are mandatory (see below).
  - If blocked by captcha/consent/login wall, stop and report clearly.

## Operating Budgets (Speed-First Defaults)

- Max queries executed: **12**
  - If a provided query pack has more than 12 queries, select the best 12 (balanced across broad/focused/methods).
- Results captured per query: **Top 5**
- Target unique papers: **15–30** total after dedupe
- If insufficient coverage after 12 queries:
  - Stop and output what you have
  - Provide up to **3 query refinements** (not additional executed queries)

## Inputs

### Minimum input (standalone mode)
- `topic` (string): the user’s research idea in plain language

### Optional input (preferred)
- `scholar_queries` (list of strings): direct copy/paste queries for Scholar

### Optional input (from Academic Scout output)
- A full “Academic Search Keyword Pack” (markdown) containing a “Google Scholar Query Combos” section.
  - The skill should attempt to extract the queries from that section.
  - If extraction fails, fall back to standalone mode.

## Standalone Query Generation (If no query list provided)

If the user does not provide `scholar_queries`, generate a compact set of Scholar queries:

1) Extract 3–6 core noun phrases from `topic` (English + Korean if relevant)
2) Generate:
   - 4 broad queries (recall)
   - 4 focused queries (precision)
   - 4 methods/measures queries (operationalization)
3) Use quotes for multi-word phrases; use OR groups sparingly.

**Important:** Do not invent named scales/instruments. Use generic measure keywords (scale, validation, measurement, psychometrics) unless a specific instrument name appears in visible sources.

## Workflow (Fast)

1) Determine execution mode:
   - If `scholar_queries` provided → use them (truncate to budget)
   - Else if Academic Scout pack provided → extract queries → use them
   - Else → generate queries from `topic`

2) Launch `browser_subagent` and navigate to:
   - `https://scholar.google.com/`

3) For each query (up to max queries):
   - Paste the query into the Scholar search box and submit
   - On the results page, capture the **top N results** (default 5), extracting:
     - Title (as shown)
     - Result link (Scholar outbound target or Scholar internal link)
     - Authors / venue / year line (as shown)
     - “Cited by X” count if visible (number only)
     - PDF link if visible (often appears on the right as [PDF])
     - Any obvious flags like [BOOK], [CITATION], [HTML] if visible
   - Record which query produced each result

4) Deduplicate papers across queries:
   - Prefer dedupe by Scholar result URL if stable
   - Otherwise dedupe by normalized title (lowercase, remove punctuation)

5) Build the output:
   - A short “What I found” summary
   - A “Top candidates” shortlist (10 items max)
   - Per-query result sections (for traceability)
   - One deduped paper table for import/triage

## Output Format (Mandatory)

```md
# Google Scholar Scout Results
- Topic: <verbatim user topic>
- Mode: <provided query pack | extracted from academic pack | standalone generated>
- Scholar queries executed (<=12):
  1) <query>
  2) <query>
  ...

## 1) Quick Summary (plain language)
- What the results suggest about the literature landscape (5 bullets max)
- Gaps / ambiguity (3 bullets max)

## 2) Top Candidates (read first)
List up to 10 papers that look most relevant/high-signal.
For each:
- [Paper Title](PRIMARY_LINK)
  - Authors / venue / year: <as shown or Unknown>
  - Cited by: <X or Unknown>
  - Full text: <PDF/HTML link if available, else Unknown>
  - Why this is a good starting point (1 line)
  - Found via: <query #>

## 3) Results by Query (traceability)
### Query 1: <query text>
1) [Title](PRIMARY_LINK)
   - Authors / venue / year: ...
   - Cited by: ...
   - Full text: ...
2) ...

### Query 2: <query text>
...

## 4) Deduped Paper Table (for triage/import)
| Paper | Authors/venue/year | Cited by | Full text | Found via | Notes |
|---|---|---:|---|---|---|
| [Title](PRIMARY_LINK) | ... | ... | ... | Q1, Q4 | (book/citation/html flag if shown) |

## 5) Query Refinements (not executed)
If coverage is weak or noisy, propose up to 3 improved queries:
- <refinement 1>
- <refinement 2>
- <refinement 3>

## 6) Failure Notes (if any)
- Captcha/consent/login wall encountered? <yes/no + where>
- Missing fields? <what>
````

## Quality Rules (Practical)

* Prefer results that look like:

  * peer-reviewed articles
  * systematic reviews / meta-analyses (when appropriate)
  * highly cited foundational papers (when relevance is clear)
* Use citation count carefully:

  * High “Cited by” can indicate foundational work, but older papers naturally accumulate more citations.
* Avoid over-indexing on patents/books unless the topic explicitly calls for it.

## Failure Modes & Safe Stop

If any of these happen, stop and return partial results:

* Google Scholar blocks with captcha
* Consent wall cannot be bypassed without account action
* Results pages fail to load repeatedly

Return what was collected so far, then:

* Report where it failed
* Suggest a smaller query set (3–5) for manual retry

## Acceptance Criteria (Done = pass)

* Uses `browser_subagent` and navigates to `https://scholar.google.com/`
* Executes no more than **12** queries
* Every query is listed in the output
* Captures **top 5** results per query (unless fewer exist)
* No invented metadata/URLs; unknown fields are labeled “Unknown”
* Provides a **deduped paper table** and a **Top Candidates** list
* Works both:

  * with a provided `scholar_queries` pack
  * with only a `topic` (standalone mode)

## Notes for Integration

This skill pairs well with:

* `google-ai-academic-scout` (keyword/query pack generation)
* Any downstream “paper reading / synthesis” agent (NotebookLM, Zotero workflows, etc.)

However, it must remain fully usable **without** upstream output.

```
```
