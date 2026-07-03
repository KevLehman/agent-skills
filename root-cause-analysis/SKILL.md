---
name: root-cause-analysis
description: Use when the user asks for a "root cause", root cause analysis, RCA, or 5 whys of a bug, incident, regression, outage, or failure — before proposing any fix or writing a postmortem.
---

# Root Cause Analysis (5 Whys)

## Overview

Find the root cause by asking "why" iteratively — each answer becomes the next question — until you reach a process or design cause that, if fixed, prevents the whole class of failure. Never stop at the first technical cause.

## The Deliverable

The output IS this chain. Not an essay, not a list of possible causes.

```
Symptom: <what was observed>
Why 1: <direct cause> — VERIFIED: <evidence> | ASSUMED: <concrete check to run>
Why 2: <cause of Why 1> — VERIFIED/ASSUMED: ...
Why N: ...
Root cause: <process / design / missing-safeguard gap>
Fix (root): <change that prevents the whole class>
Fix (symptom): <immediate mitigation, if one is needed>
Confidence: <which links are ASSUMED; what to verify before acting on this>
```

## Rules

1. Each "why" answers the line directly above it — a causal chain, not a topic list.
2. Label every link VERIFIED or ASSUMED. VERIFIED means evidence you actually saw: a quoted log line, code you read, a repro you ran, a fact stated in the report — cite it. Inference from correlation or "it must be, given the timeline" is ASSUMED, however confident you are; state the concrete check that would confirm it. If you have repo, log, or command access, run the check and upgrade the label before delivering — reading the code is cheaper than being wrong.
3. A technical cause is almost never the root. "Helper leaked connections" → why did that ship? A process/design answer (skipped review, no test, undefined requirement, missing alert) is usually one or two whys deeper. Stop only when the current answer is a fixable process/design cause, or the next why leaves anyone's control.
4. Five is a heuristic, not a quota — 3 whys or 7 whys is fine.
5. Branch when a why has two or more independent causes; follow each branch to its own root and present both chains.
6. Time pressure never shortens the chain. If evidence is missing, deliver the full chain with ASSUMED labels and their checks — a fast wrong root cause costs more than a labeled assumption.
7. A prior "fix" that didn't work is evidence: add "why was that fix believed correct?" to the chain — it usually exposes the link everyone skipped.

## Common Mistakes

- Stopping at Why 1 or 2 because it's plausible and the asker is in a hurry.
- Demoting the process cause to a "contributing factor" footnote — if fixing it prevents the class, it belongs on the chain as the root.
- Ending on a person ("engineer skipped review"). Ask why skipping was possible; root causes are systemic, not names.
- Confirming the first hypothesis instead of testing links — for each VERIFIED label, ask "what evidence would break this link?"
- Proposing the fix before the chain is complete.
