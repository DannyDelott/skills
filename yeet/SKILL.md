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
7. Open a normal pull request with `gh pr create` or the GitHub connector.
   Write its description using the requirements below. Use ready-for-review
   status by default and draft status only after an explicit draft request.
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

For a feature, the observable problem can be a missing capability. For one
prerequisite slice of a larger stack, explain both what the slice enables next
and which behavior remains for later slices.

Use this generic opening shape when useful:

```text
[Actor] currently encounters [problem] when [situation]. This happens because
[cause in everyday language]. This PR changes the flow so [concrete behavior],
preventing [specific risk]. It intentionally leaves [non-goals] unchanged.
```

Place a technical `## Summary` list such as "remove X, add Y, preserve Z" after
the causal explanation. Treat that list as an implementation inventory.
Rewrite an opening that relies on the issue, diff, or unexplained internal
names until it stands on its own.

For infrastructure, architecture, migration, or stacked slices whose value is
not directly visible, include a compact plain-English walkthrough. It must:

- define the real product entities involved before naming internal modules;
- show one concrete event flow, preferably as a short arrow chain;
- explain what capability this PR makes possible next;
- state explicitly the behavior this PR adds and the behavior left for later;
  and
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
matrices there. Omit empty sections. Make each value claim specific: replace
"improves maintainability" with the particular friction, risk, or repeated work
removed.

Before opening the PR, verify that the body starts with the causal explanation.
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
