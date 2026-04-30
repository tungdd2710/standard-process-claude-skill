---
name: standard-process
description: Structured request-handling workflow. Invoke as your FIRST action on every user request UNLESS running as Opus outside of plan mode. Step 0 triages the request (trivial/micro/continuation/substantive). Substantive requests run ANALYSE → CHECK → RESEARCH → BRAINSTORM → APPROVE → EXECUTE. Prevents scope drift, surfaces assumptions early, and ensures no code is written before research is saved.
when_to_use: At the start of every user request, before any tool calls. Skip when (a) the previous turn already triaged and this turn is clearly a direct continuation, OR (b) the active model is Opus and you are NOT in plan mode (Opus direct-execution is sufficiently thorough that ceremony adds friction; Opus plan-mode still invokes).
---

# Standard Process

Apply to ALL tasks: features, bug fixes, refactors, content, strategy, ops, lookups.

## Step 0 — Triage (always first, fast)

Classify the request, state the classification out loud, then route:

| Type | Definition | Route |
|------|------------|-------|
| **Trivial** | Yes/no, definition, meta-question, single-file lookup with answer already in context | Answer directly. Skip steps 1–6. |
| **Micro** | 1–3 line edit, single command, fix obvious typo, run one tool with known args | Skip CHECK/RESEARCH. Do step 1 (assumptions) inline + step 6 (execute). |
| **Continuation** | Follow-up clarification or next step in an ongoing task where scope is already established | Inherit triage from parent task. Do not re-triage. |
| **Substantive** | Multi-step, requires decisions, touches multiple files/systems, or user said "find ALL / research / plan / design" | Run full process steps 1–6 below. |

**Bias against under-triaging.** "Find all X" is substantive even when it sounds simple. When in doubt, treat as substantive.

State triage in one sentence so the user can correct it:
*"Triaged as substantive — running full process."*

---

## Steps 1–6 (substantive requests only)

### 1. ANALYSE
- Decompose the request into concrete sub-questions
- For EACH sub-question, classify:
  - **REUSE** — answer already exists in conversation, docs, or memory
  - **REFRESH** — prior research exists but may be stale → re-verify
  - **NEW** — no prior coverage → external research required
- For NEW/REFRESH questions, **enumerate search surfaces UP FRONT** (don't let the research phase decide scope — scope is decided here)
- State assumptions explicitly
- Output: a research brief listing sub-questions × classification × surfaces to search

### 2. CHECK EXISTING WORK
Resolve REUSE/REFRESH items by checking:
- Project research directory (e.g. `docs/research/`) — look for prior files with expiry dates
- Project index/registry document (e.g. `docs/INDEX.md`) — what's already documented
- Project progress tracker (e.g. `.claude/progress.md`, `TODO.md`) — current sprint state
- Memory files (MEMORY.md, `.claude/memory/`) — persistent feedback and project facts
- If a REUSE source is past its expiry → reclassify as REFRESH

### 3. RESEARCH EXHAUSTIVELY
Required for every non-REUSE question.
- Hit **every surface enumerated in step 1** (no scope changes during research)
- Internal first: all referenced docs, status files, architecture notes
- External: the specific surfaces named, not just "the web"
- **Save all findings to a file** (e.g. `docs/research/{TOPIC}-RESEARCH.md`) with an expiry date
- Findings must NOT live only in conversation context — they will be lost on compact

**Research surface menu** (pick the relevant ones during ANALYSE):

*Code & packages*
- GitHub topic/full-text search
- npm, PyPI, crates.io, Maven Central, pkg.go.dev

*AI / Claude-specific*
- github.com/anthropics/skills (official Claude skills)
- smithery.ai, mcp.so, mcpservers.org (MCP server marketplaces)
- Anthropic forum, Discord, changelog

*Community*
- Reddit (r/ClaudeAI, r/ClaudeCode, r/LocalLLaMA)
- Hacker News (hn.algolia.com)
- X/Twitter vendor accounts
- Vendor release notes / changelogs

*Internal*
- docs/INDEX.md, docs/research/, docs/status/
- MEMORY.md and linked memory files
- .claude/progress.md (or project equivalent)
- git log for recent decisions

**Rule:** if ANALYSE doesn't name the surfaces, RESEARCH cannot proceed — go back and decompose.

### 4. BRAINSTORM
- Present 2–3 options with concrete tradeoffs
- State a clear recommendation with rationale
- Keep options focused — don't invent options to pad the list

### 5. GET APPROVAL
- Present the plan concisely
- Do not execute until the user explicitly confirms
- If the user asks a clarifying question, answer it and re-present; don't execute yet

### 6. EXECUTE
- Update documentation first (strategy/architecture/status) — code without docs is a guess
- Then code/assets
- Update the project progress tracker
- Commit with conventional prefix (e.g. `feat:`, `fix:`, `docs:`, `chore:`)
- Log any bug fix with: problem description, commit hash, affected files, how to verify

---

## Context management

At **75% context usage**: STOP and flush before continuing.
1. Save all in-progress files to disk — no work may exist only in conversation
2. Update the project progress tracker with current state and explicit next step
3. Make the pending step list detailed enough for a fresh context to resume from

---

## When NOT to invoke this skill

- When the previous turn already completed triage and this turn is clearly the next step in the same task
- When the user types a direct command with no ambiguity (e.g. "run the tests", "commit this")
- For pure conversation (not a work request)
- **When the active model is Opus AND you are NOT in plan mode.** Opus direct-execution is thorough enough that the process ceremony adds friction without meaningful safety gain. Plan-mode Opus still invokes — plan mode is explicitly the state where structured analysis pays off. Sonnet and Haiku continue to auto-invoke in all modes.
