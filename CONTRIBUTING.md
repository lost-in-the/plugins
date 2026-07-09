# Contributing to lost-in-the/plugins

This repo is a Claude Code **marketplace** — it lists plugins; it does not host their skill
content. The skills live in their home repos (`lost-in-the/grove`, `lost-in-the/woods`) and are
referenced here via `git-subdir` sources. Keep that separation in mind: most changes to what a
plugin *does* belong in its home repo, not here.

## What lives here

- `.claude-plugin/marketplace.json` — the plugin list and their `git-subdir` sources.
- `README.md` — install UX and the min-tool-version matrix.

## The paired-PR rule (applies across all three repos)

Because a plugin's behavior is defined by a skill in another repo, changes must stay coordinated.
Before opening any PR — here or in a home repo:

1. **Documentation must be current.** Any doc affected by the change is updated in the *same*
   PR. A behavior change without its doc update is incomplete.
2. **Investigate plugin-functionality impact.** If a home-repo change touches anything a
   distributed skill relies on — a `grove` command/flag/`--json` field, a `woods` rake
   task/MCP tool/executable/config key, or setup steps — investigate whether the skill needs to
   change.
3. **Pair the PR.** If the skill must change, update it in its home repo in the same change set,
   and if the *marketplace entry* must change (new plugin, `ref`/`sha` pin, metadata), open a
   paired PR here and cross-link the two.

Each home repo's `CONTRIBUTING.md` carries the matching "Claude Code plugin changes" section.

## Changing the marketplace

- **Add a plugin:** append an entry to `plugins[]` in `marketplace.json` with a `git-subdir`
  source (`url`, `path`, optional `ref`/`sha`). The `path` must point at a valid plugin root —
  a directory whose `SKILL.md` (single-skill) or `skills/<name>/SKILL.md` (multi-skill) load,
  optionally with `.claude-plugin/plugin.json` for metadata and versioning.
- **Pin for stability:** add `"ref": "<tag>"` to a source to freeze it to a release instead of
  tracking `main`.
- **Version bumps** that users receive come from the plugin's `version` in its home-repo
  `plugin.json`, not from this repo. Don't expect a marketplace edit alone to push an update.

## Validate

```bash
# Against a local checkout of the home repo's plugin root:
claude plugin validate /path/to/grove/skills/grove-worktree-management
claude plugin validate /path/to/woods/plugin
```

Also confirm `marketplace.json` is valid JSON and each `path` resolves in the referenced repo
on the branch/ref the entry targets.
