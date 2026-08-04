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
   Write its description using the requirements below. The PR must be ready for
   review unless the user explicitly requested draft.
8. Return the PR URL, branch, commit, and checks run.

## PR Description

Open with the reason for the change in plain language. The first paragraph must
explain why the PR exists without requiring the issue or diff. Before a
technical `## Summary`, implementation bullets, file names, hashes, or test
results, explain this causal story in order:

`Observable problem -> plain-English cause -> changed behavior -> risk prevented -> explicit non-goals`

1. Describe the concrete user-visible problem. If no user sees it directly,
   name the operator or codebase situation that fails or is missing today.
2. Translate the cause into everyday language. Introduce an internal name only
   after defining what it means.
3. State what will happen instead in concrete, observable terms.
4. Name the specific harm, confusion, or failure the new behavior prevents.
5. Name adjacent behavior this PR intentionally leaves unchanged.

For a feature, the observable problem can be a capability people do not have
yet. For one prerequisite slice of a larger stack, explain both what the slice
enables next and what it deliberately does not implement.

Use this generic opening shape when useful:

```text
[Actor] currently encounters [problem] when [situation]. This happens because
[cause in everyday language]. This PR changes the flow so [concrete behavior],
preventing [specific risk]. It intentionally leaves [non-goals] unchanged.
```

Do not open with a `## Summary` list such as "remove X, add Y, preserve Z."
That is a technical inventory, not an explanation. If the opening does not
stand on its own without the issue, diff, or unexplained internal names,
rewrite it before publishing.

For infrastructure, architecture, migration, or stacked slices whose value is
not directly visible, include a compact plain-English walkthrough. It must:

- define the real product entities involved before naming internal modules;
- show one concrete event flow, preferably as a short arrow chain;
- explain what capability this PR makes possible next;
- state explicitly what this PR does and does not do yet; and
- restate the review question as one sentence a product owner can evaluate
  without codebase knowledge.

Example shape:

```text
User action -> upstream event -> this PR records or validates the missing state
-> the next feature can act on it safely
```

Use a short Before/After example when it clarifies the behavior. Define
unfamiliar domain terms when they first appear.

After the causal explanation, summarize the technical changes and verification
in concise sections. Keep hashes, file names, internal modules, and test
matrices there. Omit empty sections. Do not use generic claims such as
"improves maintainability" unless they name the specific friction, risk, or
repeated work removed.

Before opening the PR, reject any body whose opening is only a list of changes.
Confirm that a reviewer can identify the observable problem, translated cause,
changed behavior, prevented risk, and explicit non-goals without first reading
the issue or diff.

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
