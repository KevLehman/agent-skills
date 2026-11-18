---
name: filling-jira-tickets
description: Use when creating a Jira ticket in the CORE project, or filling or updating fields on an existing CORE ticket — setting priority, "Defect from", "Root Cause", "Fix versions", "Post-merge Verifier", or "Post-merge Verification" for a bug, regression, feature, tech debt, or refactor.
---

# Filling Jira Tickets (CORE)

## Overview

Every CORE ticket gets a priority from the table below, a Fix version, and either a Post-merge Verifier or Post-merge Verification "Not Applicable". Fixes (bugs and regressions) also get Root Cause, and Defect from only when it follows from that Root Cause. Never guess a value you cannot verify: ask.

Jira site: `cloudId` `6e1c19a7-7928-4dc7-aad0-584fdde3c00c`, project `CORE`.

| Field | Key | Value shape |
|---|---|---|
| Priority | `priority` | `{"name": "Highest"}` (names: Highest, High, Medium, Low, Lowest) |
| Defect from | `customfield_10086` | `[{"name": "8.6.0"}]` (version array) |
| Root Cause | `customfield_10316` | string: PR URL, or `Since creation` |
| Fix versions | `fixVersions` | `[{"name": "9.1.0"}]` |
| Post-merge Verifier | `customfield_10557` | `[{"accountId": "<accountId>"}]` (user array) |
| Post-merge Verification | `customfield_10229` | `{"value": "Not Applicable"}` (other values: `Not verified`, `Yes, it meets requirements`, `No, it does not meet requirements`; those are set by the verifier, never by you) |

## Priority

Pick the highest row that matches. "Mid" is Jira `Medium`.

| Priority | Items |
|---|---|
| Highest | Support items, critical vulnerabilities, projects, regressions |
| High | Vulnerabilities, QA fixes, features |
| Medium | Important tech debt, day-to-day tasks, improvements |
| Low | Tech debt, refactors, tests |
| Lowest | Omnichannel, everything else |

Critical vulnerability: CVSS >= 9.0; otherwise it is a plain vulnerability. Omnichannel work is Lowest only if no higher row matches (a regression in Omnichannel is still Highest).

## Fixes: Root Cause and Defect from

Finding the Root Cause usually takes a real root cause analysis, not just a `git blame` on the line the fix touches: the line that changed is often a symptom, and the introducing change can be elsewhere. If the cause is not already established, use the `root-cause-analysis` skill first, then fill the fields from its verified conclusion.

1. Find the commit that introduced the defect (`git log -S'<symbol>' -- <path>`, `git blame`).
2. Find its PR: `gh api repos/RocketChat/Rocket.Chat/commits/<sha>/pulls`.
3. Find the first release containing it: `git tag --contains <sha> --sort=version:refname`, take the earliest stable tag (no `-rc`). That tag is **Defect from**.
4. **Root Cause** is that PR's URL. If the code is from the original implementation of the feature, write `Since creation` (Defect from is then the version the feature first shipped in).
5. If you cannot trace the commit or PR, ask the user. Do not fill either field with a guess.

Defect from is only for fixes (bugs and regressions), and it is always derived from the Root Cause: it is the first stable release containing the Root Cause PR, never an independent guess. No traced Root Cause means no Defect from; leave both empty and ask.

Features, tech debt, refactors, and follow-up tasks that ship no fix: leave Defect from and Root Cause empty.

## Fix versions

The version the work is targeted to merge into:

- PR targets `develop`: the version in `apps/meteor/package.json` on `develop`, without the prerelease suffix.
- PR targets `release-X.Y.Z`: `X.Y.Z`.
- Backport: add the backport version as a second fix version, but only when the user says it is a backport.

The version name must already exist in Jira. Check it against the `fixVersions` allowed values from `getJiraIssueTypeMetaWithFields` (project `CORE`); if it is missing, ask.

## Post-merge Verifier

Suggest a verifier, and let the user choose before writing either field.

Pillar owners:

| Owner (accountId) | Areas |
|---|---|
| Owner A (`<accountId-A>`) | Area 1 keywords. Area 2 keywords |
| Owner B (`<accountId-B>`) | Area 3 keywords |
| Owner C (`<accountId-C>`) | Area 4 keywords |

Other verifiers: Verifier D (`<accountId-D>`), Verifier E (`<accountId-E>`), Verifier F (`<accountId-F>`).

Pure tests (every changed file is a test, spec, or test fixture): suggest Verifier D with Post-merge Verification `Not Applicable`, and ask the user to confirm before writing both fields. Skip the steps below.

1. Match the paths the PR (or planned fix) changes and the feature the ticket describes against the owners table.
2. Before creating or editing the ticket, ask the user in a message (not `AskUserQuestion`: it caps at 4 options). If one owner's areas match, name that owner as the suggestion, with the matching paths or feature. If none or several match (omnichannel, CI, lint, build tooling, cross-cutting refactors), say so and suggest nobody. Either way, end with a numbered list of every owner and other verifier plus "Leave empty", and wait for the answer.
3. Write `customfield_10557` with the person the user picks.
4. If the user picks "Leave empty", ask whether post-merge verification is not applicable. If yes, set `customfield_10229` to `{"value": "Not Applicable"}` and leave `customfield_10557` empty. If no, leave both fields empty and say so in the report.

## Creating

Call `createJiraIssue` with `projectKey: "CORE"` and the applicable fields in `additional_fields`: priority, Fix versions, and the Post-merge Verifier (or Post-merge Verification "Not Applicable") from the section above always; Root Cause and Defect from only for fixes with a traced Root Cause. Description rules:

- Reference tickets and PRs by Jira link and GitHub link only. Never reference Zoho tickets (public repo, customer context).
- Describe the chosen fix and link the PR, not the list of rejected options.
- No AI or Claude attribution anywhere.

## Updating an existing ticket

1. `getJiraIssue` with `fields: ["priority", "customfield_10086", "customfield_10316", "fixVersions", "issuetype", "customfield_10557", "customfield_10229"]`.
2. Edit only fields that are empty (`null` or `[]`). Treat the Post-merge Verifier as filled if either `customfield_10557` or `customfield_10229` has a value. Priority is always set on Jira tickets, so never change it on an existing ticket.
3. Report which fields were filled and which were skipped because they already had a value.
4. Apply the fills with `editJiraIssue`. If it is not available in the session, print the exact fields JSON to apply and say so. Do not create a duplicate ticket to work around it.

## Common mistakes

- Filling Root Cause from `git blame` alone, without confirming the cause (root cause analysis).
- Taking Defect from from the fix PR's release instead of the introducing PR's release.
- Setting Fix versions to the latest released version instead of the next unreleased one.
- Setting Defect from on a ticket that is not a fix, or without a Root Cause to derive it from.
- Filling Root Cause with prose instead of the PR URL.
- Overwriting a field that already has a value.
- Writing the Post-merge Verifier from the pillar match without asking the user first.
- Suggesting a verifier for a change that spans pillars or matches none.
- Setting Post-merge Verification to "Not Applicable" without asking the user first.
