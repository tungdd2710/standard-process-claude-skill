---
name: repo-eval
description: External GitHub repo evaluation — malware check, source read, license gate, cross-domain transfer test, verdict. Invoke whenever evaluating, checking, or adopting any external GitHub repo or library.
---

# External Repo Evaluation

Run every step in order. Do NOT skip to verdict without completing each gate.

## Step 1 — Malware check (FIRST, always)

```bash
gh api repos/{owner}/{repo} --jq '{stars:.stargazers_count, created:.created_at, updated:.pushed_at, archived:.archived, license:.license.spdx_id}'
gh api repos/{owner}/{repo}/contents --jq '[.[].name]'
```

STOP and flag if:
- Binary files in `src/` or `lib/` paths (`.exe`, `.dll`, `.so` outside expected build output)
- Repo <30 days old, zero stars, name matches a known package (typosquat pattern)
- `archived: true` and user hasn't acknowledged this

## Step 2 — License gate

Read the actual LICENSE file, not the README badge:

```bash
gh api repos/{owner}/{repo}/contents/LICENSE --jq '.content' | base64 -d 2>/dev/null || \
  gh api repos/{owner}/{repo}/contents/LICENSE.md --jq '.content' | base64 -d 2>/dev/null
```

Verdict:
- MIT / Apache-2.0 / BSD-* → **ADOPT** allowed (rewrite in your project's conventions, no copy-paste)
- GPL-2.0 / GPL-3.0 / AGPL → **CONCEPT-ONLY** (copyleft contaminates proprietary codebase)
- NOASSERTION / missing / README badge ≠ LICENSE file → **CONCEPT-ONLY** (vague license = zero code copy)
- Proprietary / "all rights reserved" / SSPL → **REFUSE**

## Step 3 — Read actual source (not README)

Read 2–4 representative files: the core logic module, one API handler or entry point, one test if present.

SKIP/REFUSE verdicts require code read. A verdict based on README only is invalid.

## Step 4 — Cross-domain transfer test

Ask explicitly: **What insight from this repo transfers to your project, even across domains?**

Non-obvious examples:
- Stock-analysis trend detection → user performance trajectory
- CRM follow-up scheduling → notification/reminder cadence
- SRS algorithm (Anki) → spaced-repetition for any domain

Dismissal is only valid AFTER the transfer test explicitly fails — state why it doesn't transfer.

## Step 5 — Reimplement decision

If ADOPT:
- Rewrite in your project's conventions. **Never copy code blocks** (license leak + dep drag + rebase pain).
- Log in your project's external repo registry:

```
| {owner}/{repo} | {license} | ADOPT/CONCEPT-ONLY/REFUSE | {use case} | {date} |
```

## Step 6 — Verdict

State one of:
- **ADOPT** — clean license, source read done, reimplement plan clear, logged in registry
- **CONCEPT-ONLY** — GPL/vague license OR no transferable code; describe the insight being carried over
- **REFUSE** — binary/malware risk OR proprietary OR zero transfer value; state reason
