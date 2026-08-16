---
name: help-skills
description: Use to get an overview of every skill in this package — what each one does, when to use it, whether it's explicit-only or fires automatically, and how they fit together as a pipeline. Triggers on "/help-skills", "what skills are available", "explain this skill package", "how does this pipeline work". Not to be confused with Claude Code's own built-in /help.
disable-model-invocation: true
---

# Help Skills

Explains the whole skill package on demand: what's available, what each
piece does, and how they connect — read live from each skill's own
`SKILL.md`, not a hardcoded summary that could drift out of sync as
skills are added or changed.

## Process

1. **Find every sibling skill.** Look at every other `SKILL.md` in the
   same parent `skills/` directory as this one (i.e., `../*/SKILL.md`
   relative to this file, excluding this skill itself).

2. **Read each one's frontmatter and enough of its body** to describe:
   - Its one-line purpose (from `description`)
   - Whether it's **explicit-only** (has `disable-model-invocation: true`
     — only runs when you name it directly) or **model-invoked** (can
     fire on Claude's own judgment at the relevant moment, e.g. before
     closing an issue)
   - Its role relative to the others — what feeds it, what it feeds

3. **Present a clear, scannable summary**, grouped in pipeline order
   where one exists rather than alphabetically:
   - The main sequence a project moves through
   - Any skills that act as gates or side-effects inside another skill,
     called out as such rather than listed as if they were an equal,
     separate step
   - For each skill: name, one-line purpose, explicit-only vs
     model-invoked, and roughly when to reach for it

4. **Close with the typical end-to-end flow** as a short list, and point
   to the repo's README for full detail on any individual skill.

## Tone

Concise and scannable — something to skim in a few seconds, not a copy
of every SKILL.md's full instructions. If a new skill has been added
since this description was last read, the live lookup in step 1 already
covers it — don't maintain a second, separate list anywhere.
