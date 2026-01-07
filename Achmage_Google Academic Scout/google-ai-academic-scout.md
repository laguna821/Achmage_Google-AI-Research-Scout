---
name: google-ai-research-scout
description: Rapid source-hunting workflow for compiling high-signal web sources and a NotebookLM seed pack using only https://google.com/ai via browser_subagent in default/fast mode. Use when the user asks for quick research, source collection, or a structured brief with deduped links, especially when constraints like query budgets, source hygiene, or strict output formatting are required.
---

# Google AI Research Scout

## Overview

Collect high-signal sources fast using a strict, single-tab google.com/ai workflow and produce a brief plus a NotebookLM seed pack in the required format.

## Hard Constraints (Mandatory)

- Use browser_subagent for any web search action.
- Search only inside https://google.com/ai using its on-page interface.
- Use default/fast settings only; do not click or change modes (no "Thinking", no model switches).
- Do not use the external Google search bar or any other search engine page.
- Stay in a single tab unless a link is broken.
- Never fabricate URLs, titles, dates, or claims; write "Unknown" when not found.
- Avoid category/section homepages when possible; prefer story-level pages, official docs, PDFs, standards, government/edu.
- Max 4 queries total. If still insufficient, stop and output what you have plus 2 follow-up queries.

## Workflow (Fast)

1. Confirm the user request is present and clear. If missing or ambiguous, ask a brief clarification before browsing.
2. Convert the request into 3-4 sub-queries:
   - Q1: Broad overview
   - Q2: Official or primary sources
   - Q3: Risks, criticisms, opposing view
   - Q4 (optional): Data, metrics, pricing, or timeline
3. Run the queries inside https://google.com/ai (default/fast mode) using browser_subagent:
   - Stay on one tab; open a new tab only if a link is broken.
   - Extract links from the AI answer/citations.
4. Collect candidate sources:
   - Prefer official docs, standards, government/edu, academic, or major press.
   - Avoid homepages when a specific story or document exists.
   - Record title, URL, type, and date if visible; use "Unknown" if not visible.
5. Deduplicate URLs and keep the best 8-15 sources.
6. Produce two outputs using the mandatory format.

## Output Format (Mandatory)

```md
# Initial Search Brief
- What you asked: <verbatim user request>
- 5 bullet takeaways (each bullet must cite at least 1 source link)
- What's missing / uncertain (max 5 bullets)

# Source List (deduped, 8-15)
For each source:
- [Title](URL)
  - Type: (official / academic / gov / major press / vendor / blog / community)
  - Why it matters (1 line)
  - Notes: (date if visible; any bias)

# NotebookLM Seed Pack (for the next agent)
## Suggested sources to import into NotebookLM (Top 6-10)
- <URL> - <why>

## Key themes / questions that NotebookLM should answer (10-15 items)
- <question 1>
- <question 2>
...

## Follow-up web searches (only if needed; max 2)
- <query>
- <query>
```
