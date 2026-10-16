# agent-skills

Skills for Claude Code (or any agent harness that reads `SKILL.md` files). Install by copying a skill's folder into `~/.claude/skills/`.

## Skills

### [managing-github-prs](managing-github-prs/SKILL.md)

Recipes for routine pull request work with the `gh` CLI. It covers moving your checkout onto a PR's branch without losing uncommitted work, and reading a PR's complete diff even when an output-compressing hook or the harness truncates it. It also covers listing unresolved review threads, then replying to and resolving them through GraphQL, as well as opening draft and stacked PRs, rebasing a stacked branch with `--onto` after its base is rewritten, and watching CI in the background. Each recipe lists the trap that tends to go wrong with it.

### [root-cause-analysis](root-cause-analysis/SKILL.md)

Runs a 5-whys root cause analysis when you ask for the "root cause" of a bug, incident, or failure. Instead of stopping at the first plausible technical cause, it produces an explicit why-chain — each link labeled VERIFIED (with cited evidence) or ASSUMED (with the concrete check to run) — and keeps digging until it reaches a process or design gap that, if fixed, prevents the whole class of failure. Ends with a root-level fix, a symptom-level mitigation, and a confidence summary.

### [teach-me](teach-me/SKILL.md)

Turns the agent into a tutor when you want to learn a topic rather than get a one-shot answer. It diagnoses what you know, negotiates a learning path, then teaches topic by topic with non-negotiable comprehension gates: application questions with plausible-misconception distractors, no answer reveals, no passing by elimination, and spaced recall of earlier topics. Progress persists to `~/.claude/teach-me/` so lessons resume across sessions exactly where they left off.
