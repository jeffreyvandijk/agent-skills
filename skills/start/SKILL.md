---
name: start
description: Use at the start of every new project or raw project idea — asks pointed, skeptical questions about the problem, users, scope, and constraints before any building begins, then produces a tight project brief. Triggers on "/start", "starting a new project", "kick off this idea".
disable-model-invocation: true
---

# Start

Interrogate a raw project idea until it's sharp enough to build from. The
point is not to collect trivia — it's to find the gaps, unstated
assumptions, and soft spots in the idea before time gets spent on it.

## When invoked

If the user gave an idea in the same message, start from it. If not, ask
for a one-or-two-sentence description of the idea before doing anything
else — don't guess at what they mean.

## Process

1. **Read the idea for what's missing, not just what's said.** Before
   asking anything, silently identify: what problem is this actually
   solving, who is it for, what's the smallest version that would prove
   it out, and what's the shakiest assumption in the pitch as given.

2. **Ask in small rounds, not one long form.** Two to four questions at a
   time, adapted to what's already been answered — don't run a fixed
   checklist blindly. Use AskUserQuestion for genuine multiple-choice
   trade-offs (e.g. "local tool or hosted service?"); use plain
   conversational questions for open-ended ones (e.g. "why does this need
   to exist — what's broken without it?"). Cover, as relevant:
   - **Problem & motivation** — what's broken today, why now, who's felt
     this pain
   - **Target user** — who specifically, and how is that different from
     "everyone"
   - **Scope** — what's in the first version, what's explicitly deferred
   - **Constraints** — time, tooling, budget, existing systems it has to
     fit into
   - **Success criteria** — how they'll know it worked
   - **Risks & unknowns** — the part of this they're least sure about
   - **Alternatives** — what exists already, why not use/extend that

3. **Push back on weak answers.** If an answer is vague, hand-wavy, or
   dodges the question, say so and ask again — don't let a soft answer
   pass just to keep momentum. Be skeptical, not hostile: the goal is a
   sharper idea, not a gotcha.

4. **Know when to stop.** Stop grilling once the open questions that
   actually matter for a first build are answered, or once the user
   signals they're done. Don't manufacture more rounds for their own
   sake.

5. **Close with a Project Brief**, written directly in the response:
   - **Problem** — one or two sentences
   - **Target user**
   - **Scope** — in for v1 / explicitly out
   - **Constraints**
   - **Success criteria**
   - **Open risks** — anything still genuinely unresolved
   - **Recommended next step**

   Offer to save the brief to a file in the current project if one seems
   useful (e.g. `docs/project-brief.md`) — don't create it unasked.

## Tone

Direct and a little skeptical, like a colleague doing a pre-mortem, not an
interrogator. Short questions, no padding, no restating the idea back at
length before getting to the point.
