---
name: show-me
description: Starts the local UI server after copying `.env` from the local `main` branch. Use when the user says "show me" after a change and wants the app running locally.
---

# Show Me

Use this when the user says `show me` after you have finished a change.

## Workflow

1. Go to the target repo root.
2. Overwrite the current `.env` with the copy from local `main`.
3. Start the local UI server with the repo's normal dev command.
4. Give the user the server URL to open.

## Env enforcement

- Prefer `git show main:.env > .env` from the repo root.
- Do not use remote env files or hand-written copies.
- If `main` does not contain `.env`, say so and stop.

## Server start

- Use the repo's existing local UI command.
- Prefer the package script named `dev`; otherwise use the repo's standard UI start script.
- Report the exact URL the server prints, usually `http://localhost:<port>`.
