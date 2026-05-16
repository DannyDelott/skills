---
name: update-pocock-skills
description: Sync selected mattpocock/skills global skills into this repo's root-level global skill layout. Use when user wants to update Pocock skills, compare with Matt's upstream repo, or review newly added upstream skills.
---

# Update Pocock Skills

This repo intentionally does not mirror Matt's repo layout. Matt's upstream now stores skills under bucket folders such as `skills/engineering/triage`, while this repo exposes selected global skills at the repo root:

```text
/Users/danny/Documents/projects/skills/<skill-name>/
```

`~/.agents/skills/<skill-name>` points at the repo copy, and `~/.claude/skills/<skill-name>` points at the agents symlink.

## Rules

- Fetch `upstream/main` first.
- Never run a blind merge from upstream.
- Preserve this repo's root-level skill layout.
- If an existing local skill has uncommitted edits, show the conflict/diff before overwriting.
- If upstream adds a new skill, flag it for the user instead of installing it automatically.
- Copy companion files as well as `SKILL.md` when a skill references local docs or scripts.
- After adding, removing, or renaming a global skill, verify both symlink layers with `readlink`.

## Managed upstream mappings

These are the currently selected upstream skills:

```text
design-an-interface=skills/deprecated/design-an-interface
diagnose=skills/engineering/diagnose
edit-article=skills/personal/edit-article
grill-me=skills/productivity/grill-me
grill-with-docs=skills/engineering/grill-with-docs
handoff=skills/productivity/handoff
improve-codebase-architecture=skills/engineering/improve-codebase-architecture
prototype=skills/engineering/prototype
request-refactor-plan=skills/deprecated/request-refactor-plan
review=skills/in-progress/review
tdd=skills/engineering/tdd
to-issues=skills/engineering/to-issues
to-prd=skills/engineering/to-prd
triage=skills/engineering/triage
ubiquitous-language=skills/deprecated/ubiquitous-language
write-a-skill=skills/productivity/write-a-skill
writing-beats=skills/in-progress/writing-beats
writing-fragments=skills/in-progress/writing-fragments
writing-shape=skills/in-progress/writing-shape
```

Explicitly excluded unless the user asks:

```text
caveman
setup-matt-pocock-skills
```

## Process

1. Fetch upstream:

```bash
git fetch upstream main
```

2. Check local state:

```bash
git status --short --branch
```

3. List upstream skills and compare by normalized skill name:

```bash
git ls-tree -r --name-only upstream/main |
  rg '/SKILL\.md$' |
  sed 's#/SKILL.md$##; s#.*/##' |
  sort -u
```

4. For existing mapped skills, compare before copying:

```bash
git diff --no-index -- <local-skill>/SKILL.md <(git show upstream/main:<upstream-path>/SKILL.md)
```

5. After the user approves any conflict-prone changes, copy the upstream folder into the root layout:

```bash
git archive upstream/main <upstream-path> | tar -x --strip-components=2
```

6. For renamed skills, remove the old root folder and old symlinks, then add the new symlinks:

```bash
git rm -r <old-skill>
rm -f ~/.agents/skills/<old-skill> ~/.claude/skills/<old-skill>
ln -sfn /Users/danny/Documents/projects/skills/<new-skill> ~/.agents/skills/<new-skill>
ln -sfn ../../.agents/skills/<new-skill> ~/.claude/skills/<new-skill>
```

7. Verify:

```bash
readlink ~/.agents/skills/<skill-name>
readlink ~/.claude/skills/<skill-name>
git diff --check
git status --short --branch
```
