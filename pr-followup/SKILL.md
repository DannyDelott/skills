---
name: pr-followup
description: Implement review feedback on a pull request to get it into a mergeable state. Use when user has reviewed a PR and left comments, wants to act on PR feedback, or says "follow up on PR", "implement review comments", or similar.
---

# PR Followup

Read review comments on a PR, clarify anything ambiguous, then implement all feedback. Goal: get the PR merged.

## Process

1. **Get the PR** — if the user didn't provide a PR number or URL, fetch open PRs with `gh pr list` and present them so the user can pick one.

2. **Fetch review comments** — use `gh` to get all inline and review-level comments. Identify comments from the human reviewer; skip bots and automated checks.

3. **Get context** — read the linked issue, PRD, and progress file (if present) to understand the intent behind the PR before evaluating the feedback.

4. **Clarify before coding** — identify anything ambiguous, contradictory, or unclearly scoped. Ask ALL clarifying questions in one message. Don't ask about obvious things. Wait for answers before proceeding.

5. **Check out and rebase** — check out the PR branch, fetch latest, and rebase onto the base branch. Resolve any conflicts before continuing.

6. **Implement the feedback** — work through each comment systematically. Address inline comments at the exact location referenced. Address review-level comments as broader changes. Re-read each comment after making its change to confirm it's fully addressed. Do not introduce unrelated changes.

7. **Verify** — run tests and check for type errors or lint issues. Review the diff. Fix anything you broke.

8. **Push and resolve** — commit, force-push with lease, then mark each review thread as resolved via `gh`.
