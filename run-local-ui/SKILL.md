---

name: run-local-ui
description: Starts a local UI server, reusing the app's `.env` from local `main` when the app requires one. Use when the user asks to run, open, or preview a UI locally.

---

# Run Local UI

Use when the user wants the UI running locally. If given a PR, use that branch and display it.

## Workflow

1. Find the UI app path that owns the dev server.
2. Check the app's scripts, configuration, and env examples to determine whether its local dev server requires an `.env`.
3. When required, copy that UI app's `.env` from local `main` into the same path on the current branch. It will likely not be a tracked file in git, and that is expected.
4. Start the UI with the repo's normal dev command.
5. Give the user the printed local URL.

## Env

- Prefer this command:

  ```sh
  git show main:<ui-app-path>/.env > <ui-app-path>/.env
  ```

- Do not use `main:.env > .env` unless the UI app env is actually at repo root.
- Do not copy root `.env` into a UI app.
- Do not copy UI `.env` into repo root.
- Do not use remote, generated, or hand-written env files.
- If the app requires an `.env` and it is missing from local `main`, say so and stop.
- If the app has no local env contract, continue without an `.env`; a missing file is not a blocker. For example, Windra's landing app runs without one.

## Server

- Use the existing UI dev command.
- Report the exact URL the server prints.
