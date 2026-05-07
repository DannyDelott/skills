---
name: yeet
description: Publish local changes to GitHub by staging explicit files, committing, pushing, and opening a ready-for-review pull request. Use when the user asks to yeet, ship, publish, push up, open a PR, create a pull request, or get local work into review; this local skill replaces github:yeet and must not create draft PRs unless the user explicitly asks for a draft.
---

# Yeet

Publish local work to GitHub with a normal ready-for-review PR by default.

Do not use `github:yeet` for this workflow. Do not create draft PRs unless the
user explicitly asks for a draft PR in the current request.

## Workflow

1. Inspect the worktree with `git status -sb`.
2. If the worktree has unrelated changes, stage only the files that belong to
   the requested PR. Never use `git add -A` in a mixed worktree.
3. If on `main`, `master`, or the repo default branch, create a short
   `codex/<description>` branch from the requested base branch.
4. Commit with a terse message that describes the actual diff.
5. Run the relevant checks for the changed files when practical.
6. Push the current branch with tracking.
7. Open a normal pull request with `gh pr create` or the GitHub connector.
   The PR must be ready for review unless the user explicitly requested draft.
8. Return the PR URL, branch, commit, and checks run.

## Command Shape

Prefer explicit commands:

```sh
git status -sb
git add <explicit-file> [<explicit-file> ...]
git commit -m "<message>"
git push -u origin "$(git branch --show-current)"
gh pr create --base <base> --head "$(git branch --show-current)" --title "<title>" --body-file <body-file>
```

Do not include `--draft` unless the user explicitly asks for a draft PR.

## Existing Branches

If the branch is already pushed and only needs a PR, do not recommit or amend.
Create the ready-for-review PR from the existing branch.

## Safety

- Never stage unrelated user changes silently.
- Never revert unrelated user changes.
- Never force-push unless the task is explicitly to update an existing PR branch
  and a force-push is necessary.
- If checks fail, report the failure and fix it when it is in scope.
