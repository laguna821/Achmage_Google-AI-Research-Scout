# Google AI Research Scout (FAST Scout)

**Rapid source-hunting workflow** for compiling high-signal web sources and a **NotebookLM Seed Pack** using **only** `https://google.com/ai` via `browser_subagent` in **default/fast** mode. 

## What this skill does

This skill is designed for **speed-first “initial search”**:

* Breaks a user request into **3–4 sub-queries**
* Runs them **inside `https://google.com/ai` (default/fast mode)** via `browser_subagent`
* Extracts outbound links from the AI answer/citations
* Deduplicates and keeps the **best 8–15 sources**
* Outputs:

  1. **Initial Search Brief** (5 takeaways + uncertainties)
  2. **Source List** (deduped, annotated)
  3. **NotebookLM Seed Pack** (Top sources + themes/questions + follow-up searches) 

---

## Why this exists

Many “research agents” optimize for 90–100pt quality but become **heavy and slow**. This skill intentionally targets a practical baseline: **fast recall + clean sources + structured handoff** to a deeper research step (e.g., a NotebookLM Query Pack builder).

If you need the “best possible answer,” this is not it.
If you need **a fast, usable bundle of sources + a NotebookLM-ready seed**, this is it.

---

## When to use

Use this skill when the user asks for:

* quick research / “find me sources”
* a brief with citations (links)
* “initial scan” before deeper investigation
* a NotebookLM import list + question themes for proper study 

---

## Hard constraints (mandatory)

This skill enforces strict operating rules:

* **Must** use `browser_subagent` for any web search action. 
* Search **only** inside `https://google.com/ai` using its on-page interface. 
* Use **default/fast settings only**; do **not** click or change modes (no “Thinking”, no model switches). 
* Do **not** use external Google search bar or any other search engine page. 
* **Single-tab workflow** unless a link is broken. 
* **No fabrication** (URLs/titles/dates/claims). If not found: write **“Unknown”**. 
* Avoid category/section homepages when possible; prefer story-level pages, official docs, PDFs, standards, government/edu. 
* **Query budget:** max **4 queries** total. If still insufficient, stop and output what you have + **2 follow-up queries**. 

---

## Workflow (FAST)

1. Confirm the user request is present and clear

   * If missing/ambiguous: ask one brief clarification before browsing. 
2. Convert the request into 3–4 sub-queries: 

   * Q1: broad overview
   * Q2: official/primary sources
   * Q3: risks/criticisms/opposing view
   * Q4 (optional): data/metrics/pricing/timeline
3. Run the queries inside `https://google.com/ai` (default/fast mode) via `browser_subagent`. 
4. Extract links from the AI answer/citations. 
5. Deduplicate URLs and keep the best 8–15 sources. 
6. Produce outputs using the mandatory format below. 

---

## Output format (mandatory)

The skill must output **exactly** this structure:

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


---

## NotebookLM integration (recommended)

This skill is built to hand off cleanly to a second agent (e.g., **NotebookLM Query Pack Builder**):

1) Run **FAST Scout** → get **NotebookLM Seed Pack**  
2) Import Top 6–10 sources into NotebookLM  
3) Use the “Key themes / questions” to generate a structured NotebookLM query pack  
4) Extract evidence with citations inside NotebookLM for deeper synthesis

The FAST Scout step intentionally does **not** attempt “final answers” beyond a short brief—its job is to accelerate the next stage.

---

## Repo placement

You indicated this skill lives at:

Achmage_universal_search - Lightweight/
skills/
google-ai-research-scout/
SKILL.md
README.md  <-- this file


## Contributing / modifying

If you modify the skill, keep these invariants:

- Default/fast google.com/ai only (no mode switches)
- Query budget capped at 4
- Single-tab bias
- Zero fabrication
- Mandatory output format preserved :contentReference[oaicite:20]{index=20}

---

## License

MIT License. 
laguna821@gmail.com 
