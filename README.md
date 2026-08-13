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
4. Scaffolds `README.md` (from the brief) and a stack-appropriate `.gitignore` — no `LICENSE`, no license question
5. `git init` → commit → `gh repo create --source=. --remote=origin --push`
6. Files one GitHub issue titled with the project name, body containing the full unabridged brief, with scope as a checklist
7. Reports back the repo URL and issue URL

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
  test-first via `/test`, commit, push, open a PR (`Closes #12`), and
  close the issue **only if every behavior's tests are green and the
  full suite passes** — otherwise the issue stays open with a comment
  explaining what's failing.
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

## Typical flow

```
/start       → pressure-test the idea, get a Project Brief
/setup       → turn that brief into a pushed GitHub repo + one tracking issue
/plan-work   → break that issue into sub-issues, one per scope item
/build       → implement one ticket (/build #12) or the whole backlog (/build),
                test-first via /test — issues only close once tests are green
```

## Status

Five skills built so far (`/start`, `/setup`, `/plan-work`, `/build`, `/test`). More to come as they're built.
