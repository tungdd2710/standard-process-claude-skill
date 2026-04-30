# Always-On Rules

These rules apply to every task — no invocation needed. They codify the non-obvious invariants that cause real production incidents or decision errors when violated.

Copy this file to your project root and extend it with project-specific paths and conventions.

## Evidence & Claims

Label every claim that informs a decision:

| Label | When to use |
|-------|-------------|
| `[CITED url]` | External source — URL required |
| `[VENDOR-CLAIMED]` | Vendor docs / benchmark / marketing |
| `[N=X SELF-TEST]` | Your own measurement on representative data |
| `[VERIFIED-PROD]` | Confirmed against live production system |

Rules:
- Every research finding cites a URL, doc name, or internal data source. No sourceless assertions.
- After a diagnosis, implement the fixes in the same turn (obvious fixes immediately; ask for larger changes). Reports without execution are negative-value.
- Never ship `[VENDOR-CLAIMED]` numbers as load-bearing for a production decision without a `[SELF-TEST]` to back them up.

## Security Scope (multi-tenant)

- Brand/org resolved from session token, **never** from query params or request body.
- Enforce scope strictly at every layer: middleware → server gate → API gate → DB. Never trust client-side gates alone.
- Security trumps speed, scope, and features. When in doubt, restrict.

## Multi-CLI / Parallel Agents

**Task claiming** — when 2+ Claude Code instances share the same repo, use a git-based mutex:
- Write a claim file before starting any workstream
- Release it when done
- PLAN.md status edits are invisible to other instances until committed — only claim files work

**Safe commits** — when parallel instances may have staged unrelated files, use path-isolated commits:
```bash
git commit -m "msg" -- path1 path2
```
This prevents one instance's staged files from landing in another's commit.

**Parallel background agents** — when spawning 2+ Agents that each commit+push, each MUST get its own git worktree. Shared working directory causes branch collision. Use `isolation: "worktree"` in the Agent tool call.

**Question namespacing** — when 2+ instances ask the user questions in parallel, prefix each with a topic namespace (e.g. `R-AUTH-Q1`, `R-DB-Q1`). Flat numbering (`R1`, `R2`) collides when two instances both start at 1.

## Code & Commits

- **Re-read before editing** — re-read any file immediately before editing if other agents or users may have changed it. High-traffic files (`schema`, `config`, registry files) — always re-read.
- **Typecheck before every commit** — zero type errors required. No exceptions.
- **Commit rhythm** — commit at unit-of-work boundaries (one endpoint / one page / one migration). Push when branch is >10 commits ahead of main.
- **Progress sync** — after every unit of work: update your progress tracker in the same commit.
- **No timeline estimates** — never attach weeks/months/years to plan steps. Use sequences and dependencies only (Phase 1 → Phase 2, "after X is stable"). Calendar time is meaningless without knowing competing priorities.

## Skill Routing

Add to your project's `CLAUDE.md` to activate automatic skill routing:

```markdown
## Skill routing

When the user's request matches an available skill, invoke it via the Skill
tool as your FIRST action. Do NOT answer directly, do NOT use other tools first.

Key routing rules:
- ALL tasks → invoke standard-process as FIRST action. No exceptions.
- Bugs, errors, "why is this broken" → invoke investigate
- Ship, deploy, push, create PR → invoke ship
- QA, test the site → invoke qa
- Code review → invoke review
- Save progress, checkpoint → invoke context-save
- Evaluating or adopting any external GitHub repo/library → invoke repo-eval
- Choosing between AI models, evaluating a new release → invoke model-bench
```
