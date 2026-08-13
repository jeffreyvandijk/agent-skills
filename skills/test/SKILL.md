---
name: test
description: Use before marking any issue, ticket, or piece of work as complete or closing it — verifies the work went through red-green-refactor TDD and the full suite is green before it's allowed to count as done. Also use whenever code is being written test-first. Applies automatically as part of /build's implementation step for every ticket, and is usable directly for any TDD-style coding task via /test. Triggers whenever an issue/ticket is about to be closed or marked done, or on "/test", "write this test-first", "TDD this".
---

# Test

Test-driven development, applied to every piece of implementation code
this skill touches: no production code without a preceding failing test
that justifies it, one small behavior at a time, and nothing is "done"
until the full suite is green.

## When invoked

- **Automatically, whenever an issue/ticket is about to be marked
  complete or closed** — this isn't limited to `/build`'s own flow; any
  time work is about to be reported as done, apply this process first
  rather than waiting to be asked. This skill can invoke itself on its
  own judgment (no explicit `/test` needed) specifically for this
  trigger.
- Directly, via `/test`, for any coding task that should be built
  test-first from the start.
- As part of `/build`'s implementation step — `/build` follows this
  process for every ticket it implements, and will not close an issue
  whose tests haven't gone green under it.

## Before writing any code

1. **Find the project's existing test setup.** Look for existing test
   files and config (`package.json` scripts, `pyproject.toml`,
   `pytest.ini`, `jest.config`/`vitest.config`, CI workflows, etc.) and
   match its framework and conventions. Don't introduce a second test
   framework into a project that already has one.
2. **If there's no test setup at all**, bootstrap the idiomatic default
   for the project's stack (e.g. `pytest` for Python, `vitest`/`jest`
   for JS/TS, `go test` for Go) with the minimum config needed to run a
   test. Only stop and ask if the stack itself is genuinely ambiguous
   (e.g. a mixed-language repo with no clear primary language) —
   otherwise pick the sane default and proceed.
3. **Break the work into the smallest testable behaviors.** For a
   ticket, this usually means one behavior per acceptance criterion or
   scope item, plus the edge cases and error paths it implies — not just
   the happy path.

## The cycle — repeat per behavior

1. **Red.** Write one test for the next smallest behavior, before any
   implementation code exists for it. Run it and confirm it fails, and
   fails for the *right* reason — the behavior is genuinely missing, not
   a typo, import error, or broken test setup masking the real gap.
2. **Green.** Write the minimum implementation code needed to make that
   one test pass. Don't implement behavior no test has asked for yet —
   that gets its own red step first.
3. **Refactor.** With the test green, clean up naming, structure, and
   duplication without changing behavior. Re-run the *full* test suite
   after refactoring, not just the one new test — a refactor that breaks
   something else isn't done.
4. Move to the next behavior and repeat, until every acceptance
   criterion or scope item — including edge cases and error handling —
   has its own test.

## What counts as done

- Every behavior implemented has a test that was red before it was
  green.
- The **full** test suite passes, not just what was written this
  session — a green new test next to a broken old one is a regression,
  not a pass.
- Tests are meaningful: they assert on real behavior (not tautologies
  like a mock asserted to return what it was told to return), and don't
  depend on execution order.

When invoked from `/build`: an issue is only eligible to close once this
bar is met. If it isn't — a behavior can't be made to pass, or the suite
as a whole is broken — report the failure instead of closing the issue.

## Tone

Do the work; don't narrate the cycle step-by-step in chat ("now writing
a failing test for X..." on every single line). Report the outcome.
Flag it plainly if a behavior turns out untestable as written, or if the
issue's acceptance criteria are themselves unclear — don't quietly skip
the red step just to keep moving.
