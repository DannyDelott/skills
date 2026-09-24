---
name: change-summary
description: Present a pre-implementation change summary once the approach is aligned, with a brief rationale, annotated file tree, and planned test descriptions. Use when the user wants to review what will change before implementation begins.
---

# Change summary

Make the agreed implementation concrete enough for the user to review before work begins. Use the current conversation as the scope; inspect the relevant code and test conventions to ground the proposed paths and responsibilities.

Keep this invocation read-only and present the summary in chat. Finish at the review checkpoint; begin implementation only after the user approves, or when their current instruction explicitly says to present the summary and then proceed. Earlier agreement on the approach alone is not approval to bypass this checkpoint.

## Prepare

- Reuse settled decisions. Resolve file placement and test conventions through inspection; flag only uncertainties that materially affect the implementation.
- Account for every planned addition, modification, deletion, rename, and generated output. If work already exists, distinguish it from the remaining proposal.
- Prefer the existing owners of behavior and existing test seams. Describe the smallest implementation that satisfies the agreed scope.

## Present

Start with one or two sentences explaining the concrete problem and the intended behavior after the change.

Then provide:

### Planned files

A pruned file tree rooted at the relevant repository, including only affected paths and their parent directories. Annotate every file with `add`, `modify`, `delete`, `rename`, or `regenerate`, and a short explanation of its responsibility in the change. Include test files. Show both paths for renames; label any unresolved path as tentative.

For example:

```text
repo/
└── src/
    ├── orders.ts       [modify] Reject cancellation after shipment.
    └── orders.test.ts  [modify] Cover cancellation eligibility.
```

Use the annotations as the file-by-file change list. Add prose only where a relationship or implementation decision needs explanation beyond the tree.

### Planned tests

Group by test file. For each test to add or modify, state the action and describe the scenario and expected observable result. Use concrete descriptions suitable for test names, such as “add: cancelling a shipped order returns an error and leaves the order unchanged.” Explain any planned test removal.

Separate test edits from existing checks to run. If no test edits are warranted, say so briefly and identify the appropriate existing or manual verification. Describe planned validation as future work.

### Open decisions

Include this section only for unresolved choices or assumptions that could materially change the plan. State the decision needed and its effect on files, behavior, or tests.

Keep the result proportional to the change: a brief rationale, one annotated tree, and concrete test descriptions. End by making clear that this is the proposed implementation awaiting review, unless the user already explicitly authorized continuation after the summary.
