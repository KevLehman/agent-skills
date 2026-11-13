---
name: reviewing-code
description: Use when asked to review a PR, a branch, the current diff, or your own changes before committing — "review this", "code review", "check my diff", "is this ready", "/reviewing-code". Also use when acting as the reviewer subagent for an implementation task.
---

# Reviewing Code

## Overview

Two passes over the same diff: the built-in `code-review` skill for correctness bugs, then the house checklist below for what that skill does not look for. One combined report at the end.

A finding is reported only after you have read the code that proves it. A rule match alone is a hypothesis.

## Pass 1: correctness

Invoke the built-in `code-review` skill (Skill tool) with the user's effort level and target. If the user gave none, pass the target alone. Keep its verified findings; do not post them yet.

If the target is a PR, read its full diff the way `managing-github-prs` describes (`rtk proxy git diff origin/<base>...HEAD` after a fetch). A summarized or truncated diff is not a review.

## Pass 2: house checklist

Walk the diff once per row. For each hit, open the surrounding code and confirm it before recording it.

| Area | Flag when |
|---|---|
| New endpoints | For each new route, list its callers, the service function it wraps, and any other route wrapping the same function. If another route or an earlier/later step in the same client flow already returns the data or runs the check, flag it as removable and name the replacement. |
| Reuse | New helper, hook, model query, or array/set utility when the codebase, a shared package, or the stdlib already has one. Search with `rg` before claiming; cite the existing symbol. |
| Abstraction | A helper with exactly one caller; a wrapper around another wrapper; params, defaults or fields no caller uses; an interface with one implementation. Recommend inlining. |
| Data layer | A query filter or update document built outside the model layer; validation or input guards inside a model method (the caller validates). Iterating with `await cursor.toArray()` then `.map`/`.filter` when the transform fits on the cursor. |
| Async | An async function ending in `await x();` with nothing after it and no enclosing `try` — should be `return x();`. |
| Control flow | `=== 0` / `> 0` / `!== undefined` used only as an emptiness check where truthy/falsy is equivalent; `let` + `if` where `const x = cond && value` reads cleaner; guards nested inside unrelated guards; single-line unbraced `if (…) return`. |
| Over-guarding | Defensive checks on internal, already-typed values. Validation belongs at trust boundaries only. |
| Comments | Any comment that narrates the mechanism just written, restates a name, or explains design a rename could carry. Keep only facts external to the code (domain rules, workarounds for a named external bug). Same bar for JSDoc. |
| Tests | Tautological tests (asserting a mock was called with the args the code visibly passes, a hardcoded mapping maps, or anything that cannot fail unless the test setup is wrong); one-line request helpers abstracted in tests; `if (x) { cleanup }` guards that silently skip; leftover `console.log`; many one-assertion tests that should be one `it` or `it.each`; a new spec file for a UI component that had none. |
| Scope | Hunks unrelated to the PR's stated concern (autofix fallout, drive-by refactors). On a release branch: anything beyond the minimal regression fix. |
| Migrations | A database migration in a non-major release. Stale data stays; note it for the next major. |
| Edition boundary | Community code importing enterprise code, or an ad-hoc registry for cross-edition hooks where the repo already has a patch/override mechanism. |
| Project rules | Anything the repo's own `CLAUDE.md`, `AGENTS.md`, or `docs/` guidelines forbid. Read the ones matching the touched area before this pass. |

## Report

One list, most severe first. Each finding:

- `path:line` — one-sentence defect
- why it matters, in one line
- the concrete fix, naming the existing symbol to reuse when there is one

Correctness findings from pass 1 rank above checklist findings of the same severity. Drop anything you could not confirm by reading the code. Render PR numbers and ticket keys as markdown links.

If the user asks to post the review, post it as written — no AI attribution, no internal or customer ticket references on a public repo.

## Common mistakes

| Mistake | Fix |
|---|---|
| Running only the checklist | Pass 1 first; the checklist does not hunt bugs |
| Flagging "reuse X" without finding X | `rg` the symbol and cite its path |
| Reporting a rule hit without context | Open the file; many hits are legitimate (e.g. `=== 0` where `0` is a valid value) |
| Reviewing a summarized diff | `rtk proxy git diff …`, read overflow files fully |
| Endpoint looks fine in isolation | Trace the client flow; redundancy only shows across steps |
