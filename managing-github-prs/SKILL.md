---
name: managing-github-prs
description: Use when asked to move to or check out a PR's branch, review a PR's full diff, answer or resolve PR review comments (cubic, CodeRabbit, reviewers), open a draft or stacked PR, keep a stacked branch in sync with its base, or watch a PR's CI — in a repo using the gh CLI.
---

# Managing GitHub PRs

## Overview

Recipes for the PR chores that come up every session. Each one has a trap that cost real time; the trap is listed with the recipe.

Before any git operation, run `git branch --show-current`. Run each command alone: never append `echo` or status checks to it.

## Move to a PR's branch

"Move to this branch" means the user's own checkout. Never use a worktree unless asked.

```bash
gh pr view <N> --json headRefName,baseRefName --jq '{headRefName,baseRefName}'
git status --short                                     # dirty tracked files?
git stash push -m "<branch>: <what>" -- <files>        # only if dirty; tell the user
git worktree list                                      # branch held by a worktree blocks checkout
git checkout <headRefName> && git pull --ff-only
```

- If `pull --ff-only` fails, the remote branch was rewritten. Run `git cherry -v origin/<b> <b>`: lines starting with `-` are already on origin. Before any `reset --hard origin/<b>`, show the user the `+` commits and wait for their OK, or keep them first with `git branch backup/<b> <b>`.
- A branch checked out in another worktree can't be checked out here. Remove that worktree first, if it's clean.
- `git worktree add -B <b>` silently resets an existing local branch. Check `git branch --list <b>` first.
- End your reply with the restore step: `git stash pop` on the original branch.

## Read the complete diff

```bash
git fetch origin <base> <head>        # a stale base ref silently widens the diff to old commits
rtk proxy git diff origin/<base>...HEAD -- . ':(exclude,glob)**/*.spec.*' ':(exclude,glob)**/*.test.*'
```

- If an output-compressing hook such as rtk is installed, plain `git diff` comes back summarized, ending with `(more changes truncated)`. Prefix the command with `rtk proxy` to get the raw output. Without such a hook, drop the prefix.
- When output is too large, the harness saves it to a file and shows a 2KB preview. Read that file in chunks. The preview alone is not a review.

## Review threads: list, reply, resolve

Only GraphQL exposes `isResolved` and the thread ID (`PRRT_…`). cubic's login is `cubic-dev-ai`.

```bash
gh api graphql -F owner='{owner}' -F name='{repo}' -F pr=<N> -f query='query($owner:String!,$name:String!,$pr:Int!){repository(owner:$owner,name:$name){pullRequest(number:$pr){reviewThreads(first:100){nodes{id isResolved isOutdated path line comments(first:5){nodes{author{login} body}}}}}}}' \
  --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved|not) | {id,path,line,author:.comments.nodes[0].author.login,body:.comments.nodes[0].body}'

reply_resolve() {
  gh api graphql -f query='mutation($id:ID!,$body:String!){addPullRequestReviewThreadReply(input:{pullRequestReviewThreadId:$id,body:$body}){comment{id}}}' -f id="$1" -f body="$2" --jq '.data.addPullRequestReviewThreadReply.comment.id' &&
  gh api graphql -f query='mutation($id:ID!){resolveReviewThread(input:{threadId:$id}){thread{isResolved}}}' -f id="$1" --jq '.data.resolveReviewThread.thread.isResolved'
}
reply_resolve PRRT_xxx "Fixed in <sha>."
```

Verify every bot claim against the code before answering. Replies are one line:
- fixed: "Fixed in `<sha>`."
- already deferred in the PR description: point to that section
- declined: the reason

## Open a draft PR (stacked or not)

```bash
git push -u origin <branch>
gh pr create --draft --base <base-branch> --head <branch> --title "<type>: …" --body "$(cat <<'EOF'
## Proposed changes (including videos or screenshots)
## Issue(s)
## Steps to test or reproduce
## Further comments
EOF
)"
gh pr view <N> --json commits,changedFiles --jq '{commits:(.commits|length),changedFiles}'
```

Use the section headings from the repo's `.github/PULL_REQUEST_TEMPLATE.md`; the four above are one example. Always `--draft`. Leave out AI and Co-Authored-By lines. In a public repo, leave out internal or customer ticket references too. If the commit and file counts are far above what you changed, the base moved under you. See the next section.

## Keep a stacked branch in sync

```bash
git fetch origin <base> <branch>
git log --oneline origin/<base>..HEAD        # should list only this branch's commits
git cherry -v origin/<base> <old-base-ref>   # '-' = already in base under a new SHA
git rebase --onto origin/<base> <old-base-sha> <branch>
git push --force-with-lease origin <branch>
```

- If the base was rebased or squash-merged, a plain `git rebase origin/<base>` replays the old base commits. Use `--onto` with the old base SHA.
- After a push, GitHub recomputes PR counts lazily. Re-run `gh pr view` before you trust them.
- Sync means rebase, never merge.

## Watch CI

```bash
gh pr checks <N>                                   # snapshot
gh pr checks <N> --watch --interval 60             # Bash run_in_background, timeout up to 7200000
```

- Pushing restarts CI.
- A check whose link is the bare `https://github.com/` belongs to an external bot and has no logs. Say so; don't guess at the failure.

## Common mistakes

| Mistake | Fix |
|---|---|
| Worktree when the user said "move to the branch" | Check out in the main repo; stash dirty files and say so |
| Reviewing an rtk-summarized diff | `rtk proxy git diff …`; read saved overflow files fully |
| Resolving a thread with REST | GraphQL `resolveReviewThread` with the `PRRT_` ID |
| Stacked PR shows dozens of commits | Base was rewritten: `rebase --onto origin/<base> <old-base-sha>` |
| Trusting PR counts right after a push | Re-query; GitHub recomputes lazily |
