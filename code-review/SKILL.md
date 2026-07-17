---
name: code-review
description: Review the changes since a fixed point (commit, branch, tag, or merge-base) along two correctness axes — Standards and Spec — plus a reviewability profile and, when needed, a semantic PR split. Runs the correctness reviews in parallel sub-agents and reports them side by side. Use when the user wants to review a branch, a PR, work-in-progress changes, or asks to "review since X".
---

Two-axis review of the diff between `HEAD` and a fixed point the user supplies:

- **Standards** — does the code conform to this repo's documented coding standards?
- **Spec** — does the code faithfully implement the originating issue / PRD / spec?

Both axes run as **parallel sub-agents** so they don't pollute each other's context, then this skill aggregates their findings.

The parent review also produces a **reviewability profile**. Reviewability is
not a third correctness axis and does not mask Standards or Spec findings. It
describes the cognitive shape of the change and, when necessary, proposes a
stack of smaller PRs. Review remains read-only; use `/split-pr` to perform an
approved split.

The issue tracker should have been provided to you — run `/setup-matt-pocock-skills` if `docs/agents/issue-tracker.md` is missing.

## Process

### 1. Pin the fixed point

Whatever the user said is the fixed point — a commit SHA, branch name, tag, `main`, `HEAD~5`, etc. If they didn't specify one, ask for it.

Capture the diff command once: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base). Also note the list of commits via `git log <fixed-point>..HEAD --oneline`.

Before going further, confirm the fixed point resolves (`git rev-parse <fixed-point>`) and the diff is non-empty. A bad ref or empty diff should fail here — not inside two parallel sub-agents.

### 2. Identify the spec source

Look for the originating spec, in this order:

1. Issue references in the commit messages (`#123`, `Closes #45`, GitLab `!67`, etc.) — fetch via the workflow in `docs/agents/issue-tracker.md`.
2. A path the user passed as an argument.
3. A PRD/spec file under `docs/`, `specs/`, or `.scratch/` matching the branch name or feature.
4. If nothing is found, ask the user where the spec is. If they say there isn't one, the **Spec** sub-agent will skip and report "no spec available".

### 3. Identify the standards sources

Anything in the repo that documents how code should be written, such as `CODING_STANDARDS.md` or `CONTRIBUTING.md`.

On top of whatever the repo documents, the Standards axis always carries the **smell baseline** below — a fixed set of Fowler code smells (_Refactoring_, ch.3) that applies even when a repo documents nothing. Two rules bind it:

- **The repo overrides.** A documented repo standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **Always a judgement call.** Each smell is a labelled heuristic ("possible Feature Envy"), never a hard violation — and, like any standard here, skip anything tooling already enforces.

Each smell reads *what it is* → *how to fix*; match it against the diff:

- **Mysterious Name** — a function, variable, or type whose name doesn't reveal what it does or holds. → rename it; if no honest name comes, the design's murky.
- **Duplicated Code** — the same logic shape appears in more than one hunk or file in the change. → extract the shared shape, call it from both.
- **Feature Envy** — a method that reaches into another object's data more than its own. → move the method onto the data it envies.
- **Data Clumps** — the same few fields or params keep travelling together (a type wanting to be born). → bundle them into one type, pass that.
- **Primitive Obsession** — a primitive or string standing in for a domain concept that deserves its own type. → give the concept its own small type.
- **Repeated Switches** — the same `switch`/`if`-cascade on the same type recurs across the change. → replace with polymorphism, or one map both sites share.
- **Shotgun Surgery** — one logical change forces scattered edits across many files in the diff. → gather what changes together into one module.
- **Divergent Change** — one file or module is edited for several unrelated reasons. → split so each module changes for one reason.
- **Speculative Generality** — abstraction, parameters, or hooks added for needs the spec doesn't have. → delete it; inline back until a real need shows.
- **Message Chains** — long `a.b().c().d()` navigation the caller shouldn't depend on. → hide the walk behind one method on the first object.
- **Middle Man** — a class or function that mostly just delegates onward. → cut it, call the real target direct.
- **Refused Bequest** — a subclass or implementer that ignores or overrides most of what it inherits. → drop the inheritance, use composition.

### 4. Assess reviewability

Read any repository rule that limits PR size. Measure the diff against the
fixed point with `git diff --numstat <fixed-point>...HEAD`. Follow the
repository's exact counting and exclusion rules; do not invent exclusions. If
the rule counts added lines only, use the first `--numstat` column and ignore
deletions. Record every excluded file and why it was excluded.

If the measured diff exceeds a documented hard limit, add a hard violation to
the Standards report. A working branch may exceed a publication limit when the
repository allows it; the violation means the branch must be split before it is
published as a PR.

Build a reviewability profile without collapsing it into a numeric complexity
score:

- **Review size** — counted lines, limit, and excluded files.
- **Review questions** — the independent behavioural or architectural claims a
  reviewer must evaluate.
- **Domain spread** — modules and ownership boundaries touched.
- **Semantic density** — mechanical, behavioural, or mixed.
- **Operational risk** — migrations, authorization, money, concurrency,
  deployment sequencing, or other production-sensitive behaviour.
- **Dependency shape** — standalone, stacked, or expand-migrate-contract.

If the change exceeds a hard limit or bundles multiple review questions,
propose the smallest coherent PR stack. Each proposed PR must:

- center on one review question;
- satisfy the repository's size rule;
- be independently mergeable and green;
- declare its dependencies and verification;
- preserve complete vertical slices where possible.

For a wide mechanical change that cannot land as vertical slices, use an
expand-migrate-contract sequence. Never split by arbitrary file groups or line
ranges.

### 5. Spawn both sub-agents in parallel

Send a single message with two `Agent` tool calls. Use the `general-purpose` subagent for both.

**Standards sub-agent prompt** — include:

- The full diff command and commit list.
- The list of standards-source files you found in step 3, **plus the smell baseline from step 3** pasted in full — the sub-agent has no other access to it.
- The brief: "Report — per file/hunk where relevant — (a) every place the diff violates a documented standard: cite the standard (file + the rule); and (b) any baseline smell you spot: name it and quote the hunk. Distinguish hard violations from judgement calls — documented-standard breaches can be hard, but baseline smells are always judgement calls, and a documented repo standard overrides the baseline. Skip anything tooling enforces. Under 400 words."

**Spec sub-agent prompt** — include:

- The diff command and commit list.
- The path or fetched contents of the spec.
- The brief: "Report: (a) requirements the spec asked for that are missing or partial; (b) behaviour in the diff that wasn't asked for (scope creep); (c) requirements that look implemented but where the implementation looks wrong. Quote the spec line for each finding. Under 400 words."

If the spec is missing, skip the Spec sub-agent and note this in the final report.

### 6. Aggregate

Present the two reports under `## Standards` and `## Spec` headings, verbatim or lightly cleaned. Prepend any deterministic PR-budget violation to `## Standards`. Do **not** merge or rerank findings — the two axes are deliberately separate (see _Why two axes_).

Then add `## Reviewability` with the profile from step 4. If a split is needed,
add `### Suggested PR stack`; for each PR list its review question, contents,
dependency, approximate counted additions, and independent verification. If no
split is needed, say so explicitly.

End with a one-line summary: total findings per correctness axis, the worst issue _within each axis_ (if any), and whether the current diff is publishable as one PR under the repository's reviewability rules. Don't pick a single winner across axes — that's the reranking the separation exists to prevent.

## Why two axes

A change can pass one axis and fail the other:

- Code that follows every standard but implements the wrong thing → **Standards pass, Spec fail.**
- Code that does exactly what the issue asked but breaks the project's conventions → **Spec pass, Standards fail.**

Reporting them separately stops one axis from masking the other.
