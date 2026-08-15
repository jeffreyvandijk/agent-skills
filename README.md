# agent-skills

Custom [Claude Code](https://claude.com/claude-code) skills.

## What's a skill?

A skill is a packaged set of instructions Claude Code can invoke for a specific
kind of task (e.g. a deploy checklist, a repo-specific workflow, a review
process). Skills live under `skills/`, one directory per skill:

```
skills/
  my-skill/
    SKILL.md        # required: instructions + YAML frontmatter
    references/      # optional: supporting docs the skill can load
    scripts/          # optional: helper scripts the skill can run
```

`SKILL.md` starts with YAML frontmatter naming the skill and describing when
it should trigger:

```markdown
---
name: my-skill
description: One-line description of what this does and when to use it.
---

Instructions for Claude to follow when this skill is invoked.
```

Claude Code auto-discovers skills placed in a recognized skills directory and
lists them as invocable via `/my-skill` or by name.

## Installing these skills

Clone this repo somewhere, then make its `skills/` contents visible to
Claude Code one of these ways:

### Personal (all your projects) — recommended

Symlink each skill into your personal skills directory so every project
on your machine gets them:

```bash
git clone https://github.com/jeffreyvandijk/agent-skills.git ~/projects/agent-skills
mkdir -p ~/.claude/skills
for d in ~/projects/agent-skills/skills/*/; do
  ln -sfn "$d" ~/.claude/skills/"$(basename "$d")"
done
```

Symlinking (rather than copying) means `git pull` in `~/projects/agent-skills`
keeps every installed skill up to date. Picked up automatically — no
restart needed for personal skills already being watched.

### Project-only

To scope these skills to a single project instead of your whole machine,
symlink (or copy) them into that project's `.claude/skills/` directory:

```bash
mkdir -p /path/to/your-project/.claude/skills
for d in ~/projects/agent-skills/skills/*/; do
  ln -sfn "$d" /path/to/your-project/.claude/skills/"$(basename "$d")"
done
```

If `.claude/skills/` didn't exist in that project before you added it,
restart Claude Code once so it starts watching the new directory.

### Ad hoc, without installing

Add this repo's `skills/` folder to a running session for the current
window only, without copying or symlinking anything:

```
/add-dir ~/projects/agent-skills
```

### Plugin marketplace

This repo is also a Claude Code plugin marketplace (`.claude-plugin/marketplace.json`
at the root), bundling all 7 skills as one plugin since they're an
interdependent pipeline, not standalone tools:

```
/plugin marketplace add jeffreyvandijk/agent-skills
/plugin install agent-skills@agent-skills
```

Skills installed this way are namespaced (`/agent-skills:start`, etc.) and
don't conflict with the manual symlink method above — both can be used at
once if you want.

## Skills

### `/start`

Interrogates a raw project idea before any building begins. Asks pointed,
skeptical questions in small rounds — problem & motivation, target user,
scope, constraints, success criteria, risks, alternatives — and pushes back
on vague answers instead of letting them slide. Closes with a Project
Brief (problem, target user, scope, constraints, success criteria, open
risks, recommended next step).

Requires explicit invocation (`disable-model-invocation: true`) — it won't
fire on its own judgment, only when you run `/start`.

See [`skills/start/SKILL.md`](skills/start/SKILL.md).

### `/setup`

Takes the Project Brief from `/start` and turns it into a real GitHub repo:

1. Proposes a kebab-case repo name from the brief, confirms name/casing with you
2. Asks visibility (public/private) — never assumes
3. Picks a local directory (defaults to `~/projects/<repo-name>`, refuses to overwrite an existing path)
4. Scaffolds a **short** `README.md` (name, one-line description, scope summary — not the full brief) and a stack-appropriate `.gitignore` — no `LICENSE`, no license question
5. `git init` → commit → `gh repo create --source=. --remote=origin --push`
6. Files one GitHub issue titled with the project name, body containing the full unabridged brief, with scope as a checklist — **the single source of truth**, not duplicated into the README
7. Links the README to that issue and pushes the small follow-up commit
8. Reports back the repo URL and issue URL

Always creates under the personal GitHub account (no org prompt). Also
requires explicit invocation via `/setup`.

See [`skills/setup/SKILL.md`](skills/setup/SKILL.md).

### `/plan-work`

Takes the one big tracking issue from `/setup` and breaks it into real
GitHub sub-issues:

1. Fetches the parent issue and extracts its scope checklist, one candidate sub-issue per item
2. Shows the proposed breakdown and waits for confirmation/edits before creating anything
3. Creates each sub-issue with `gh issue create`
4. Links each one to the parent via GitHub's native sub-issue relationship (`gh api .../sub_issues`, since `gh` has no `--parent` flag)
5. Leaves the parent issue open as the epic — GitHub renders its sub-issue progress automatically
6. Reports back the parent issue URL and all new sub-issue URLs

Also requires explicit invocation via `/plan-work`.

See [`skills/plan-work/SKILL.md`](skills/plan-work/SKILL.md).

### `/build`

Implements the sub-issues from `/plan-work` as real code. Two modes:

- **`/build #12`** — implement exactly that issue: branch, implement
  test-first via `/test`, commit, push, open a PR (`Closes #12`). Merges
  the PR and closes the issue only after clearing **two gates**: (1)
  `/test`'s TDD bar — every behavior's tests are green and the full suite
  passes, and (2) `/review`'s two-axis code review comes back with no
  confirmed findings. Either gate failing leaves the issue open with a
  comment explaining what's blocking it. Once closed, runs `/finish`
  against the parent epic in case that was the last sub-issue.
- **`/build`** (no number) — implement every open, buildable issue in the
  repo (skipping parent/epic issues). Only runs from an effectively fresh
  session — refuses if the conversation already has substantial unrelated
  history, since it needs the room. Confirms the full issue list with you
  first, then works through them **one at a time, each in its own fresh
  subagent** so no ticket's implementation context leaks into the next.
  Reports a summary table (issue → PR + closed, or left open with why) at
  the end.

Also requires explicit invocation via `/build`.

See [`skills/build/SKILL.md`](skills/build/SKILL.md).

### `/test`

Test-driven development, enforced at every step: no implementation code
without a preceding failing test that justifies it, one small behavior
(including edge cases and error paths) at a time, red → green → refactor,
full suite green before anything counts as done.

Used three ways:

- **Model-invoked automatically whenever an issue/ticket is about to be
  marked complete or closed** — this is the only one of these skills
  Claude can trigger on its own judgment, since the whole point is that
  it applies before closing work, not just when explicitly asked.
- Directly via `/test`, for any coding task that should be built
  test-first from the start.
- Inside `/build`'s implementation step — every ticket `/build`
  implements goes through this process, and **an issue can only be
  closed once its tests have gone green under this discipline.**

See [`skills/test/SKILL.md`](skills/test/SKILL.md).

### `/review`

A two-axis code review, run as two independent subagents in parallel
against a diff:

- **Correctness subagent** — logic errors, mishandled edge cases,
  crashes, behavior that doesn't match the issue's acceptance criteria.
- **Simplification/efficiency subagent** — duplication, unnecessary
  complexity, missed opportunities to reuse existing code, inefficient
  patterns.

Both subagents verify each finding against the actual code before
reporting it — no speculative flags, and an empty list is a valid clean
result. Findings are merged and posted as a single PR comment.

Used three ways, same pattern as `/test`:

- **Model-invoked automatically whenever an issue/ticket is about to be
  marked complete or closed.**
- Directly via `/review`, to review any diff/PR/branch on demand.
- Inside `/build`'s implementation step, as the second closing gate,
  after `/test`'s TDD gate passes — **any confirmed finding blocks the
  issue from closing.**

See [`skills/review/SKILL.md`](skills/review/SKILL.md).

### `/finish`

Closes the loop `/setup` opened. A sub-issue closing doesn't mean the
project is done — the parent epic only closes once every sub-issue
`/plan-work` created under it is closed too.

- Checks every sibling sub-issue's state via the sub-issues API.
- If any sibling is still open, does nothing — reports how many are done
  and stops. No partial close, no nagging comment.
- If every sibling is closed, closes the parent epic with a rollup
  summary linking each sub-issue's merged PR.

Used two ways, same pattern as `/test`/`/review`:

- **Model-invoked automatically whenever a sub-issue is closed** — most
  often right after `/build` finishes a ticket.
- Directly via `/finish`, to check or close any epic on demand.

See [`skills/finish/SKILL.md`](skills/finish/SKILL.md).

## Typical flow

```
/start       → pressure-test the idea, get a Project Brief
/setup       → turn that brief into a pushed GitHub repo + one tracking issue
/plan-work   → break that issue into sub-issues, one per scope item
/build       → implement one ticket (/build #12) or the whole backlog (/build),
                test-first via /test, merged + closed only after /review comes back clean
/finish      → once the last sub-issue closes, closes the parent epic with a rollup summary
```

## Status

Seven skills built so far (`/start`, `/setup`, `/plan-work`, `/build`, `/test`, `/review`, `/finish`). More to come as they're built.
