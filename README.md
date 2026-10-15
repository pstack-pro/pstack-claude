# pstack-claude

A Claude Code plugin port of [pstack](https://github.com/cursor/plugins/tree/main/pstack) (poteto's rigorous agent workflow). It uses Claude models only (fable / opus / sonnet / haiku) and never calls external models or CLIs.

## Prerequisites

- [Claude Code](https://code.claude.com) (tested on 2.1.289)
- Node.js 18 or later (the plugin's hooks are Node scripts)

## Install / try / uninstall

Run these from the root of the project where you want pstack:

```bash
# Enable for this project only (writes .claude/settings.local.json, which is gitignored by default)
claude plugin marketplace add pstack-pro/pstack-claude --scope local
claude plugin install pstack@pstack-claude --scope local

# Update to the latest version on GitHub
claude plugin marketplace update pstack-claude

# Uninstall from this project
claude plugin uninstall pstack@pstack-claude --scope local
claude plugin marketplace remove pstack-claude --scope local
```

With a local-scope install, `claude plugin list` in other projects shows pstack as disabled; it isn't loaded there.

To hack on the code, or to try it for a single session, clone it anywhere and use the local path:

```bash
git clone https://github.com/pstack-pro/pstack-claude.git <any-dir>

# Try it once: loaded for this session only, no config written
claude --plugin-dir <any-dir>

# Or install from the local directory into this project (Claude Code stores an absolute path)
claude plugin marketplace add <any-dir> --scope local
claude plugin install pstack@pstack-claude --scope local
```

**On a given machine, pick one source: GitHub or a local directory.** The marketplace name `pstack-claude` is global. Adding it again from the other source replaces the source for every project, and removing it deletes that global entry, breaking pstack in other projects. To switch sources, uninstall from every project that has pstack enabled, then reinstall from the new source.

## Getting started

1. `/pstack:setup-pstack`: writes `.pstack/models.md`, starting from the `claude-tiers` mapping; you can change any role.
2. `/pstack:poteto-mode <task>`: the main entry point; it picks the right playbook automatically.

The `claude-tiers` mapping:

| Role | Model |
|---|---|
| hardest tasks | `fable` |
| Judgment and writing (judgment, how/why synthesis, reflect review) | `opus` |
| Coding (feature, bug-fix, perf, refactoring, swarm) | `sonnet` |
| Investigation (how explorer, why investigators) | `haiku` |
| Review panels (arena, architect, interrogate) | `opus, sonnet, haiku` |

Reasoning effort isn't set per role; it follows the session's `/effort`. After editing `.pstack/models.md`, run `/clear` or start a new session so the new config gets injected.

If you don't want to commit the model config to your project repo, add `.pstack/` to the project's `.gitignore`.

## Differences from the Cursor version

The converted skill bodies are kept essentially unchanged; the gaps are filled by:

- `CLAUDE-CODE.md`: maps Cursor tools and features to Claude Code one by one — e.g. `Task` → `Agent`, `readonly` → `pstack:readonly`, `environment: "cloud"` → `isolation: "worktree"`, `AskQuestion` → `AskUserQuestion`. The main session gets it via a SessionStart hook; pstack's agents and poteto-mode read it themselves.
- `hooks/session-start.mjs`: injects that guide, `.pstack/models.md`, and the current transcript path into new sessions.
- `hooks/allow-plugin-read.mjs`: auto-approves Read, Glob, and Grep on files inside the plugin directory, since skills need to read their own playbooks. Paths outside the plugin directory, and symlinks pointing outside it, still go through the normal permission flow.
- `agents/readonly.md`: a general-purpose agent with editing tools disabled, equivalent to Cursor's `readonly: true`.
- Config lives in `.pstack/models.md` rather than under `.claude/`, because Claude Code treats `.claude/` as sensitive and asks for confirmation on every write.

## Tracking upstream

```bash
node scripts/sync.mjs                        # shallow-clones github.com/cursor/plugins into the system temp dir
node scripts/sync.mjs <plugins-repo>/pstack   # or point at an existing local checkout
PSTACK_SRC=<plugins-repo>/pstack node scripts/sync.mjs

test/smoke.sh                                # offline checks: no model calls, no usage
```

`sync.mjs` regenerates `skills/` and `agents/`, then layers `overrides/` on top (`setup-pstack` and the `readonly` agent). The upstream version is recorded in `UPSTREAM.json`.

## License

pstack was created by Lauren Tan and is MIT-licensed; see `LICENSE-pstack`.

## Related repositories

- Chinese edition (skill descriptions and bodies translated to Chinese, preview): <https://github.com/pstack-pro/pstack-claude-zh>
- The other port: <https://github.com/pstack-pro/pstack-pi>
- Upstream pstack: <https://github.com/cursor/plugins/tree/main/pstack>
