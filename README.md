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

## Typical flow

```
/start   → pressure-test the idea, get a Project Brief
/setup   → turn that brief into a pushed GitHub repo + one tracking issue
```

## Status

Two skills built so far (`/start`, `/setup`). More to come as they're built.
