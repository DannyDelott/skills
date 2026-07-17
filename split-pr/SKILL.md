---
name: split-pr
description: Split an oversized or cognitively dense branch into a stack of focused, reviewable pull requests. Use when a code review recommends a PR split, a branch exceeds its repository's PR budget, a change bundles multiple review questions that should land separately, or the user asks to process PRs labeled `oversized:flagged-for-research`.
---

# Split PR

Turn a completed working branch into a stack of focused PRs. Preserve the
source branch as the complete reference implementation.

## Research queue

Treat `oversized:flagged-for-research` as the inbox for PRs that may need
splitting, and `oversized:umbrella` as the confirmed umbrella state for a
tracked split.

When asked to process the research queue, inspect open PRs labeled
`oversized:flagged-for-research` one at a time. Run `/code-review` against the
PR's base when it has no current reviewability report, then update the original
PR's durable `<!-- split-pr-tracker -->` comment with the finding and proposed
stack, creating that comment when it does not exist.

- If no split is needed, explain why in the comment and remove
  `oversized:flagged-for-research`.
- If the split needs approval, leave `oversized:flagged-for-research` applied
  and mark the comment `Awaiting approval`.
- If the user already authorized splitting any PR that needs it, or approves
  the plan, replace `oversized:flagged-for-research` with
  `oversized:umbrella` and continue through this skill.

Keep the original PR open in every branch of this workflow.

## Review questions

A **review question** is the main claim a reviewer must evaluate to approve a
PR. Derive it from the spec and diff; it is not a fixed checklist. Common shapes
include:

- **Behaviour** — does this slice deliver the intended user or system behaviour?
- **Contract** — is a public API, schema, or interface correct and compatible?
- **Architecture** — is responsibility placed behind the right boundary?
- **Migration** — can callers or data move safely from the old form to the new?
- **Operations** — can the change be deployed, observed, and rolled back safely?
- **Removal** — is the old path unused and safe to delete?
- **Mechanical** — is a rename, move, or retype complete without changing behaviour?

A PR may contain supporting work, but it should have one primary review
question. Closely coupled questions may stay together when separating them
would create an unsafe or unverifiable intermediate state.

## Process

### 1. Understand the complete change

Record the source branch, source SHA, base branch, and merge base. Read the
originating issue or spec, repository standards, and the latest `/code-review`
report when one exists. Inspect the full diff and measure it using the
repository's PR-budget rules.

Keep the source branch unchanged so the complete implementation remains
recoverable throughout the split. When it already has an oversized PR, keep
that PR open as the umbrella for the split.

### 2. Design the stack

List the review questions in the complete change, then group the diff into the
smallest useful set of PRs. Each proposed PR must:

- have one primary review question;
- satisfy the repository's PR budget;
- be green and mergeable at its declared base;
- include the code, tests, and generated artifacts needed for its behaviour;
- leave the repository in a useful, verifiable state.

Prefer complete vertical slices. For a wide mechanical change that cannot land
vertically, use expand-migrate-contract: add the new form, migrate callers in
reviewable batches, then remove the old form.

### 3. Confirm the plan

Present the proposed PRs in landing order. For each, show its review question,
contents, base or dependency, counted additions, and verification.

Get the user's approval before creating or rewriting branches or PRs.

Once approved, replace `oversized:flagged-for-research` with
`oversized:umbrella` on the original PR, or add `oversized:umbrella` when the
research label was not present. Create either label when it is missing and the
repository permits it. Post or update one tracking comment on the original PR
using this shape:

```md
<!-- split-pr-tracker -->
## Split tracking

**Status:** In progress
**Why this was split:** <reviewability summary>
**Source issue/spec:** <links>

| Slice | Review question | Issue | PR | Status |
| ----- | --------------- | ----- | -- | ------ |
| 1     | ...             | ...   | ... | Planned |

**Landing order:** ...
```

Keep this as one durable comment: find it by its marker and edit it as the split
progresses instead of posting disconnected progress comments.

### 4. Carve and verify

Create a separate branch or worktree for each approved PR. Build the first from
the original base and each dependent PR from its approved parent. Prefer whole
existing commits; otherwise select and edit the necessary changes until the
slice matches the approved boundary.

For every branch:

1. Review its diff against its declared base.
2. Confirm it satisfies the repository's PR budget.
3. Run the relevant tests, typechecking, linting, and build checks.
4. Commit only when the slice is coherent and green.
5. Update the original PR's tracking comment with its branch, issue, PR, and
   current status.

If a slice cannot be made independently safe, revise the stack instead of
weakening verification or forcing an arbitrary split.

### 5. Publish when requested

When the user asks for publication, push and open the PRs in dependency order.
State each PR's review question, parent, verification, and landing order in its
description. Keep the original oversized PR open and leave its
`oversized:umbrella` label applied unless the user explicitly asks to close it.
Update its tracking comment with every child link and mark the split complete
when all planned PRs are available.

Finish when the source remains recoverable and every carved PR is within
budget, green at its declared base, and explicit about its dependency and
review question, and the original PR accurately shows the split's plan and
progress.
