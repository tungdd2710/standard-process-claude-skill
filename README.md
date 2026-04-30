# claude-process-skills

> **A collection of structured workflow skills for Claude Code.**

Three skills + a set of always-on rules that prevent the most common Claude Code failure modes: scope drift, lost research, unsourced claims, unsafe parallel execution, and code written without approval.

---

## Skills

### `standard-process` — Triage + 6-step workflow

Stop Claude from diving straight into code before it understands the problem.

Every request gets classified first (trivial / micro / continuation / substantive), and substantive work follows a fixed sequence:

```
1. ANALYSE   — decompose into sub-questions; name research surfaces upfront
2. CHECK     — hit existing docs, progress tracker, memory before any new search
3. RESEARCH  — exhaustively cover every named surface; save findings to files
4. BRAINSTORM — 2–3 options with tradeoffs; recommend one
5. APPROVE   — no code without explicit confirmation
6. EXECUTE   — update docs first, then code, then commit
```

**Without this skill**, Claude tends to over-execute simple requests, under-execute complex ones, lose research to context compaction, and surprise you with sweeping changes when you asked a question.

**Opus exception:** when running as Opus *outside* plan mode, the skill auto-skips — Opus direct-execution is sufficiently thorough that the ceremony adds friction without safety gain. Opus in plan mode still invokes.

---

### `repo-eval` — External repo evaluation

Before adopting any external GitHub repo or library, run this 6-step gate:

1. **Malware check** — metadata + directory listing; flag binaries in `src/`, typosquats, archived repos
2. **License gate** — read the actual LICENSE file (MIT/Apache → ADOPT, GPL → CONCEPT-ONLY, vague → CONCEPT-ONLY, proprietary → REFUSE)
3. **Source read** — 2–4 representative files; SKIP/REFUSE invalid without code read
4. **Cross-domain transfer test** — explicitly ask what transfers to your project, even across domains
5. **Reimplement decision** — rewrite in your conventions; never copy code (license leak + dep drag + rebase pain)
6. **Verdict** — ADOPT / CONCEPT-ONLY / REFUSE with stated reason

---

### `model-bench` — AI model benchmarking

Before switching models in production, run this 7-step gate:

1. **Catalog freshness** — 3-way diff: your host ↔ provider library ↔ OpenRouter; pull missing tags first
2. **Candidate slate** — bench ≥3 (open-source self-host + hosted commercial + cheap/fast); open-source wins ties
3. **Jurisdiction gate** — flag restricted-jurisdiction paid endpoints; self-host is always OK
4. **Dataset license gate** — LLM-distilled datasets inherit source AUP; refuse them
5. **Run bench** — label everything `[N=X SELF-TEST]`; never ship `[VENDOR-CLAIMED]` as load-bearing
6. **P-Vault threshold** — <50% on canonical metric → one re-audit → vault per-task if still failing
7. **Spec card + fallback** — update AI feature registry; previous winner stays as fallback (never drop)

---

## Always-On Rules (`CLAUDE.md`)

The `CLAUDE.md` in this repo contains rules that need no invocation — they apply to every task. Copy it to your project root and extend it:

- **Evidence & Claims** — label every claim: `[CITED]` / `[VENDOR-CLAIMED]` / `[N=X SELF-TEST]` / `[VERIFIED-PROD]`
- **Security Scope** — session-scoped auth; defense-in-depth at every layer
- **Multi-CLI / Parallel Agents** — git-based task claiming, path-isolated commits, worktrees for parallel agents
- **Code & Commits** — re-read before edit, typecheck before commit, commit at unit-of-work boundaries

---

## Install

```bash
git clone https://github.com/tungdd2710/standard-process-claude-skill
cp -r standard-process-claude-skill/skills/* ~/.claude/skills/
cp standard-process-claude-skill/CLAUDE.md ./CLAUDE.md  # or merge into your existing one
```

Then add skill routing to your `CLAUDE.md`:

```markdown
## Skill routing

When the user's request matches an available skill, invoke it via the Skill
tool as your FIRST action.

Key routing rules:
- ALL tasks → invoke standard-process as FIRST action. No exceptions.
- Evaluating or adopting any external GitHub repo/library → invoke repo-eval
- Choosing between AI models, evaluating a new release → invoke model-bench
```

---

## Versioning

`plugin.json` version is automatically bumped (patch) on every push that touches `skills/**` or `CLAUDE.md`, via the GitHub Actions workflow in `.github/workflows/semver-bump.yml`.

For significant changes (new skill = minor, breaking change = major), edit `plugin.json` manually before pushing — the action will then bump patch from your new base.

---

## Design principles

- **No overhead on simple requests.** Trivial and micro triage adds zero friction.
- **Scope is declared, not discovered.** ANALYSE names surfaces before research starts — makes Claude auditable.
- **Work survives context compaction.** All research goes to files in step 3. Sessions compress; knowledge doesn't.
- **Approval is a hard gate.** Step 5 is not a suggestion. No execution without an explicit yes.
- **Portable.** Nothing project-specific. Customize via `CLAUDE.md` overrides, not by editing skills.

---

## License

MIT
