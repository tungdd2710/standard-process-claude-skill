# standard-process — Claude Code Skill

A structured request-handling workflow for Claude Code. Every request gets triaged into one of four types, and substantive work follows a 6-step process that prevents scope drift, surfaces assumptions early, and ensures research is saved before code is written.

## What it does

**Step 0 — Triage** (always runs first):

| Type | Definition |
|------|------------|
| Trivial | Yes/no, definition, single lookup — answer directly |
| Micro | 1–3 line edit, obvious typo — do it inline |
| Continuation | Follow-up in an ongoing task — inherit triage |
| Substantive | Multi-step, decisions, multiple files — run full process |

**Steps 1–6** (substantive only): ANALYSE → CHECK → RESEARCH → BRAINSTORM → APPROVE → EXECUTE

Key properties:
- Research scope is named upfront in ANALYSE, not discovered during research
- All findings are saved to files before code is written (no work lost to context compaction)
- No code is written without explicit user approval

## Install

### As a Claude Code plugin (recommended)

```bash
# Coming soon — publish to plugin marketplace
```

### Manual install (works today)

```bash
# Copy the skill to your user skills directory
cp -r skills/standard-process ~/.claude/skills/
```

Then invoke it with `/standard-process` in any Claude Code session.

### Project-level (committed to repo)

```bash
# Copy to .claude/skills/ in your project
cp -r skills/standard-process /path/to/your/project/.claude/skills/
```

Note: Project-level `.claude/skills/` is **not** auto-loaded by Claude Code at this time. Use the user-level `~/.claude/skills/` install instead, or reference the skill file from your `CLAUDE.md` via a `@` path.

## Usage

In Claude Code, the skill auto-triggers as the first action on every request (when `description` and `when_to_use` fields match). You can also invoke it explicitly:

```
/standard-process
```

## Customizing for your project

After installing, you can override the internal document paths to match your project structure by adding a note to your `CLAUDE.md`:

```markdown
## Standard Process — path overrides

When running standard-process steps, use these project-specific paths:
- Research directory: `notes/research/`
- Project index: `docs/INDEX.md`  
- Progress tracker: `.claude/progress.md`
- Memory: `.claude/memory/MEMORY.md`
```

## Contributing

Issues and PRs welcome. The SKILL.md is intentionally generic — project-specific conventions belong in CLAUDE.md, not in this skill.

## License

MIT
