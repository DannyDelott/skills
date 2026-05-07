---

name: show-me
description: Starts the local UI server after copying the UI `.env` from local `main`.

---

# Show Me

Use when the user says `show me` and wants the UI running locally. If given a PR, use that branch and display it.

## Workflow

1. Find the UI app path that owns the dev server.
2. Copy that UI app's `.env` from local `main` into the same path on the current branch. It will likely not be a tracked file in git, and that is expected.
3. Start the UI with the repo's normal dev command.
4. Give the user the printed local URL.

## Env

- Prefer this command:

  ```sh
  git show main:<ui-app-path>/.env > <ui-app-path>/.env
  ```

- Do not use `main:.env > .env` unless the UI app env is actually at repo root.
- Do not copy root `.env` into a UI app.
- Do not copy UI `.env` into repo root.
- Do not use remote, generated, or hand-written env files.
- If the UI `.env` is missing from local `main`, say so and stop.

## Server

- Use the existing UI dev command.
- Report the exact URL the server prints.
