# Repository Instructions

This repo is the source of truth for shared skills.

- Authoritative skill sources live in `/Users/danny/Documents/projects/skills/<skill-name>/`
- `~/.agents/skills/<skill-name>` points at the repo copy
- `~/.claude/skills/<skill-name>` points at `~/.agents/skills/<skill-name>`

To add a new global skill:

1. Create the skill folder in this repo with a `SKILL.md`.
2. Add the matching symlink in `~/.agents/skills/`.
3. Add the matching symlink in `~/.claude/skills/`.
4. Verify the links with `readlink`.
5. Commit the change in this repo.
6. Push `main` to your fork so GitHub stays in sync.
