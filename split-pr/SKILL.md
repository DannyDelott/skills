---
name: split-pr
description: Split an oversized or cognitively dense branch into a stack of focused, reviewable pull requests. Use when a code review recommends a PR split, a branch exceeds its repository's PR budget, or a change bundles multiple review questions that should land separately.
---

# Split PR

Carve a completed working branch into focused branches that can be reviewed and
landed safely in dependency order. Preserve the source branch as the complete
reference implementation.

## Process

### 1. Pin the source and base

Record the source branch, its HEAD, the intended base branch, and the merge
base. Require a clean worktree before changing branches. Fetch the remote refs
without rebasing, resetting, amending, or force-pushing the source branch.

Read the originating issue or spec, repository coding standards, and the latest
`/code-review` report when one exists. If the report contains a suggested PR
stack, treat it as the starting hypothesis rather than an instruction to copy
blindly.

This step is complete when the original implementation can always be recovered
from the recorded source branch and SHA.

### 2. Map the review questions

Inspect `git diff <merge-base>...<source>` and the source branch's commits.
Measure additions using the repository's exact PR-budget and exclusion rules.

Name each independent behavioural or architectural claim as a **review
question**. Build a dependency graph whose nodes are proposed PRs. Each node
must:

- answer one coherent review question;
- fit the repository's PR budget;
- be green and mergeable at its declared base;
- include the tests and generated artifacts needed for that question;
- expose a useful, verifiable state after it lands.

Prefer complete vertical slices. For a wide mechanical change that cannot land
vertically, use expand-migrate-contract: add the new form, migrate callers in
reviewable batches, then remove the old form. Do not divide work by arbitrary
file groups, equal line counts, or code without its tests.

### 3. Confirm the stack

Present the proposed branches in landing order. For each branch, show:

- review question;
- declared base and dependencies;
- included behaviour;
- approximate counted additions;
- verification that will prove it independently green.

Ask for confirmation before creating, rewriting, pushing, closing, or retargeting
branches or PRs. Revise the graph until every node has a coherent boundary and
the user approves it.

### 4. Carve without rewriting the source

Create a separate branch or worktree for every approved node, following the
repository's branch naming rules. Build the first node from the original base;
build each dependent node from its approved parent branch.

Prefer whole existing commits when they already match a node. Otherwise apply
commits or hunks without committing, then edit the result until it forms the
approved complete slice. Preserve authorship where practical. Never reset or
force-push the source branch merely to manufacture the stack.

After carving each node:

1. Review its diff against its declared base.
2. Measure it against the repository's PR budget.
3. Run its relevant tests, typechecking, linting, and build checks.
4. Commit only when the slice is coherent, within budget, and green.

If a node cannot meet all three constraints without an unsafe intermediate
state, stop and redesign the dependency graph rather than weakening tests or
hiding required code in another PR.

### 5. Publish only when requested

If the user requested publication, push the branches and open PRs in dependency
order. Target the repository's normal base for the first PR and the approved
parent branch for each dependent PR. In every PR body, state:

- its review question;
- its parent PR or branch;
- its counted additions and exclusions;
- its verification;
- the landing order and any retargeting needed after a parent merges.

Do not close, replace, retarget, or force-update an existing oversized PR unless
the user approved that action. When approved, mark it as superseded only after
the replacement stack is available.

If publication was not requested, stop with clean local branches and report the
commands or branch order needed to publish them later.

## Completion

Finish only when:

- the recorded source branch and SHA remain recoverable;
- every carved PR answers one review question;
- every carved PR satisfies its repository's budget;
- every carved PR is green at its declared base;
- dependencies and landing order are explicit;
- all created worktrees are clean;
- published PR URLs are returned when publication was requested.
