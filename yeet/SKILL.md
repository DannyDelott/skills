---
name: yeet
description: Publish local changes to GitHub by staging explicit files, committing, pushing, and opening a ready-for-review pull request. Use when the user asks to yeet, ship, publish, push up, open a PR, create a pull request, or get local work into review; this local skill replaces github:yeet and uses draft status only when the user explicitly asks for a draft.
---

# Yeet

Use this local workflow instead of `github:yeet`. Publish local work to GitHub
with a normal ready-for-review PR by default. Reserve draft status for an
explicit draft request in the current conversation.

## Workflow

1. Inspect the worktree with `git status -sb`.
2. If the worktree has unrelated changes, stage only the requested files by
   explicit path.
3. If on `main`, `master`, or the repo default branch, create a short
   `codex/<description>` branch from the requested base branch.
4. Commit with a terse message that describes the actual diff.
5. Run the relevant checks for the changed files when practical.
6. Push the current branch with tracking.
7. Call the Skill tool with "pr" and draft the pull request description using
   its Summary, Evidence, and Merge Danger format.
8. Treat the complete draft as the last message and call the Skill tool with
   "wait-what". Replace the draft with its re-pitch, then confirm the re-pitch
   preserved the `pr` structure, evidence, and scope boundaries.
9. Open a normal pull request with `gh pr create` or the GitHub connector using
   the re-pitched description. Use ready-for-review status by default and draft
   status only after an explicit draft request.
10. Return the PR URL, branch, commit, and checks run.

## PR Description

`pr` owns the description format. Keep the concrete reason for the change clear
within its Summary, use only observed evidence, and disclose validation limits.
Use `wait-what` to simplify the language while preserving that structure.

## Command Shape

Prefer explicit commands:

```sh
git status -sb
git add <explicit-file> [<explicit-file> ...]
git commit -m "<message>"
git push -u origin "$(git branch --show-current)"
gh pr create --base <base> --head "$(git branch --show-current)" --title "<title>" --body-file <body-file>
```

Include `--draft` only after an explicit request for a draft PR.

## Existing Branches

If the branch is already pushed and only needs a PR, preserve its existing
commits and create the ready-for-review PR from that branch.

## Safety

- Stage only explicitly scoped files and report any unrelated user changes.
- Preserve unrelated user changes.
- Use a normal push by default. Reserve force-push with lease for an explicit
  request to update an existing PR when the branch history requires it.
- If checks fail, report the failure and fix it when it is in scope.
