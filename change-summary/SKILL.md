---
name: change-summary
description: Review the agreed implementation before work begins, with a brief rationale, annotated file tree, and planned test descriptions.
disable-model-invocation: true
---

# Change summary

Turn the current agreement into a concise implementation proposal in chat. This is the user's review checkpoint before editing.

## Ground the proposal

Use the conversation for scope and settled decisions. Inspect the relevant files and test conventions until every proposed change has a concrete location and responsibility. Label unresolved paths as tentative. Distinguish any existing work from the remaining proposal.

## Present the summary

**Why:** Start with one or two sentences explaining the concrete problem and intended behavior after the change.

**Planned files:** Show a pruned file tree containing every affected file and its parent directories, including tests and generated outputs. Annotate each file with the action (`add`, `modify`, `delete`, `rename`, or `regenerate`) and a one-line description of the change. Show both paths for renames. Let the tree serve as the change list; add prose only for relationships or decisions the annotations cannot explain.

**Planned tests:** Group by test file. For every test to add or modify, give the action, scenario, and expected observable result. For example: “add: cancelling a shipped order returns an error and leaves the order unchanged.” Explain any test removals. Separately list existing checks or manual verification to run. If no test edits are warranted, briefly explain why.

Mention open decisions only when they materially affect files, behavior, or tests.

## Finish at the checkpoint

The summary is ready when every planned file change is represented and every proposed test describes observable behavior. Keep preparation read-only and end awaiting the user's review. Begin implementation after approval, or when the current instruction explicitly says to summarize and then proceed; agreement on the approach alone does not bypass this checkpoint.
