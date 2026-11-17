---
name: filling-jira-tickets
description: Use when creating a Jira ticket in the CORE project, or filling or updating fields on an existing CORE ticket — setting priority, "Defect from", "Root Cause", or "Fix versions" for a bug, regression, feature, tech debt, or refactor.
---

# Filling Jira Tickets (CORE)

## Overview

Every CORE ticket gets a priority from the table below and a Fix version. Fixes (bugs and regressions) also get Root Cause, and Defect from only when it follows from that Root Cause. Never guess a value you cannot verify: ask.

Jira site: `cloudId` `6e1c19a7-7928-4dc7-aad0-584fdde3c00c`, project `CORE`.

| Field | Key | Value shape |
|---|---|---|
| Priority | `priority` | `{"name": "Highest"}` (names: Highest, High, Medium, Low, Lowest) |
| Defect from | `customfield_10086` | `[{"name": "8.6.0"}]` (version array) |
| Root Cause | `customfield_10316` | string: PR URL, or `Since creation` |
| Fix versions | `fixVersions` | `[{"name": "9.1.0"}]` |

## Priority

Pick the highest row that matches. "Mid" is Jira `Medium`.

| Priority | Items |
|---|---|
| Highest | Support items, critical vulnerabilities, projects, regressions |
| High | Vulnerabilities, QA fixes, features |
| Medium | Important tech debt, day-to-day tasks, improvements |
| Low | Tech debt, refactors |
| Lowest | Omnichannel, everything else |

Critical vulnerability: CVSS >= 9.0; otherwise it is a plain vulnerability. Omnichannel work is Lowest only if no higher row matches (a regression in Omnichannel is still Highest).

## Bugs and regressions: Defect from and Root Cause

1. Find the commit that introduced the defect (`git log -S'<symbol>' -- <path>`, `git blame`).
2. Find its PR: `gh api repos/RocketChat/Rocket.Chat/commits/<sha>/pulls`.
3. Find the first release containing it: `git tag --contains <sha> --sort=version:refname`, take the earliest stable tag (no `-rc`). That tag is **Defect from**.
4. **Root Cause** is that PR's URL. If the code is from the original implementation of the feature, write `Since creation` (Defect from is then the version the feature first shipped in).
5. If you cannot trace the commit or PR, ask Kevin. Do not fill either field with a guess.

Defect from is only for fixes (bugs and regressions), and it is always derived from the Root Cause: it is the first stable release containing the Root Cause PR, never an independent guess. No traced Root Cause means no Defect from; leave both empty and ask.

Features, tech debt, refactors, and follow-up tasks that ship no fix: leave Defect from and Root Cause empty.

## Fix versions

The version the work is targeted to merge into:

- PR targets `develop`: the version in `apps/meteor/package.json` on `develop`, without the prerelease suffix.
- PR targets `release-X.Y.Z`: `X.Y.Z`.
- Backport: add the backport version as a second fix version, but only when Kevin says it is a backport.

The version name must already exist in Jira. Check it against the `fixVersions` allowed values from `getJiraIssueTypeMetaWithFields` (project `CORE`); if it is missing, ask.

## Creating

Call `createJiraIssue` with `projectKey: "CORE"` and the four fields in `additional_fields`. Description rules:

- Reference tickets and PRs by Jira link and GitHub link only. Never reference Zoho tickets (public repo, customer context).
- Describe the chosen fix and link the PR, not the list of rejected options.
- No AI or Claude attribution anywhere.

## Updating an existing ticket

1. `getJiraIssue` with `fields: ["priority", "customfield_10086", "customfield_10316", "fixVersions", "issuetype"]`.
2. Edit only fields that are empty (`null` or `[]`). Priority is always set on Jira tickets, so never change it on an existing ticket.
3. Report which fields were filled and which were skipped because they already had a value.
4. Apply the fills with `editJiraIssue`. If it is not available in the session, print the exact fields JSON to apply and say so. Do not create a duplicate ticket to work around it.

## Common mistakes

- Taking Defect from from the fix PR's release instead of the introducing PR's release.
- Setting Fix versions to the latest released version instead of the next unreleased one.
- Setting Defect from on a ticket that is not a fix, or without a Root Cause to derive it from.
- Filling Root Cause with prose instead of the PR URL.
- Overwriting a field that already has a value.
