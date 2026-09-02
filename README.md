# lost-in-the/plugins

A Claude Code **marketplace suite** for the `lost-in-the` tools. It hosts multiple plugins from
one place, each referencing its home repo's skill subtree via a **`git-subdir`** source — so
installing a plugin fetches only the skill files (a sparse partial clone), not the entire tool
repo.

| Plugin | What it gives you | Home repo | Min tool version |
|--------|-------------------|-----------|------------------|
| `grove-plugin` | The `grove-worktree-management` skill — command reference, safety rules, and deterministic helper scripts for [grove](https://github.com/lost-in-the/grove), the git worktree + tmux manager. | [`lost-in-the/grove`](https://github.com/lost-in-the/grove) → `skills/grove-worktree-management/` | grove **≥ 0.8.0** |
| `woods-plugin` | Five guide skills — `woods-setup`, `woods-mcp-config`, `woods-investigate`, `woods-agent-enable`, `woods-diagnose` — for [woods](https://github.com/lost-in-the/woods), the Rails code-intelligence gem: install/upgrade, MCP configuration, index-driven investigation (audits, code reviews, impact analysis), repository agent enablement, and diagnosis. | [`lost-in-the/woods`](https://github.com/lost-in-the/woods) → `plugin/` | woods **≥ 2.0.0** |

## Install

```bash
# In Claude Code:
/plugin marketplace add lost-in-the/plugins
/plugin install grove-plugin@lost-in-the-plugins
/plugin install woods-plugin@lost-in-the-plugins
```

Update later with `/plugin update grove-plugin@lost-in-the-plugins` (same for `woods-plugin`).

## Why a suite (and why it's lean)

Previously grove shipped its own repo-root marketplace (`source: "./"`), so installing cached
the **entire grove repo** (~4 MB — Go sources, tests, TUI fixtures, a demo GIF) into
`~/.claude/plugins/cache`, ~50× more than the ~80 KB of skill files that are actually used.

`git-subdir` fixes this: Claude Code does a sparse, partial clone of just the referenced
subdirectory. The skill files stay in their home repo (single source of truth — no
duplication), and only that subtree is fetched.

## Version compatibility

The skills instruct agents to run `grove` / `woods` commands. A user can have an **older** tool
installed than the plugin assumes, so every skill opens with a **Version Preflight**:

- It captures the installed version (`grove version`; `bundle info woods`) and instructs the
  agent to operate **only** against capabilities that version actually has — never to invoke a
  command, flag, or tool the installed version lacks.
- If a workflow needs something newer, the agent tells the user to update rather than guessing
  at syntax. `grove --check-update` surfaces available grove releases (it runs even under
  agent mode, which otherwise suppresses the passive update notice).

The **Min tool version** column above is the floor each plugin currently assumes. When a plugin
starts documenting a newer capability, bump both the floor here and the plugin's
`version` (see Maintenance), and land it with the tool release that ships the capability.

## Structure

```
lost-in-the/plugins/
├── .claude-plugin/marketplace.json   # grove-plugin + woods-plugin (git-subdir sources)
├── README.md
└── CONTRIBUTING.md                    # marketplace maintenance + the paired-PR rule
```

The plugin *content* lives in the home repos, not here:

- `grove-plugin` → `lost-in-the/grove` at `skills/grove-worktree-management/` (manifest at
  `.../.claude-plugin/plugin.json`; `SKILL.md` at the root loads as a single-skill plugin).
- `woods-plugin` → `lost-in-the/woods` at `plugin/` (manifest at
  `plugin/.claude-plugin/plugin.json`; five skills under `plugin/skills/`).

## Maintenance

- **Sources track each home repo's default branch (`main`).** The `git-subdir` entries omit a
  `ref`, so they resolve against `main`. To pin a plugin to a specific release for stability,
  add `"ref": "<tag>"` (and optionally `"sha": "<commit>"`) to that entry's `source`.
- **Updates are driven by the plugin `version`** in each home repo's `plugin.json`. Bump it on
  every skill release; pushing skill changes without bumping `version` is a no-op for users who
  already have that version cached.
- **Changes that touch a home repo's skill** must follow that repo's `CONTRIBUTING.md` Pre-PR
  requirements (docs current + plugin-impact investigated + paired PR). See
  [CONTRIBUTING.md](CONTRIBUTING.md).
- Validate before publishing: `claude plugin validate` against a local checkout of each home
  repo's plugin root.
