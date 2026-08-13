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

## Status

Scaffold only for now — skills will be added as they're built.
