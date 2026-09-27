# claude-config

Version-controlled Claude Code configuration. This repository is the canonical
producer for the rendered `~/.claude` target; it is consumed by `xnoto/dotfiles`
as a main-branch chezmoi archive external, not cloned or applied directly.

| File | Deployed to | Purpose |
| --- | --- | --- |
| `CLAUDE.md` | `~/.claude/CLAUDE.md` | Global instructions, MCP routing, skill selection and execution boundaries |
| `settings.json` | `~/.claude/settings.json` | User-scope settings, permissions, plugins |
| `mcp.json` | `~/.claude/mcp.json` | MCP servers, loaded via `--mcp-config` |
| `skills/context7/SKILL.md` | `~/.claude/skills/context7/SKILL.md` | On-demand library documentation workflow |
| `skills/context-mode-routing-policy/SKILL.md` | `~/.claude/skills/context-mode-routing-policy/SKILL.md` | Bounded context-mode use without bypassing dedicated tools or approvals |

Repository tooling (`README.md`, `LICENSE`, `.gitignore`,
`.pre-commit-config.yaml`, `.secrets.baseline`) is excluded from the archive and
never lands in `~/.claude`.

`~/.claude` belongs to Claude Code, which keeps unmanaged runtime state there
(`projects/`, `sessions/`, `plugins/`, `history.jsonl`). This repository
contributes files to that directory; it does not own it. See the `[".claude"]`
external in `xnoto/dotfiles` for the constraints that follow from that.

## Context skills

The two user skills are instruction-only. `context7` is loaded for relevant
library documentation; `context-mode-routing-policy` is loaded before substantial
output processing or context-mode execution. `CLAUDE.md` retains the load-bearing
safety rules even before a skill is loaded and provides a relative-file fallback
if skill discovery is unavailable.

The existing `context-mode@context-mode` plugin is already enabled in
`settings.json`, pinned through its marketplace to `v1.0.169`. Its
[plugin manifest](https://github.com/mksglu/context-mode/blob/589d8214d56740a28b5f7bf63167743d586b0b40/.claude-plugin/plugin.json)
loads its own skills, including `context-mode`. The distinct local name
`context-mode-routing-policy` avoids shadowing that upstream skill. It supplements
routing and approval policy, not the plugin's implementation or hooks. Native
[user-skill discovery](https://code.claude.com/docs/en/skills) uses
`~/.claude/skills`; the current archive mapping includes these files without a
dotfiles change. This repository owns its native instructions; they are not
generated or automatically synchronized from `opencode-config`.

This change does not alter plugin versions, MCP entries, permissions, hooks,
packages, or agent definitions. The existing bare context-mode executable and
remote Context7 connection still need their normal runtime prerequisites. Missing
tools must produce a reported limitation and bounded read-only fallback, not an
automatic installation, upgrade, or configuration repair.

Pre-commit currently checks JSON syntax and secret detection; there is no GitHub
Actions workflow or skill-runtime test. Static review does not establish plugin
installation, skill discovery, hook behavior, or MCP health. After approved source
publication and owner-run installation, a new client session and controlled
routing/discovery checks remain separate verification steps. Do not run
installation, service actions, or credentialed probes merely to validate source.

## Deploying a change

1. Commit and push here.
2. Run `make install` in `xnoto/dotfiles`.

Nothing in `xnoto/dotfiles` needs editing. Its `[".claude"]` external tracks
this repository's `main` branch, so `main` is the deploy boundary: anything
pushed here ships on the next apply.

The external carries no commit pin and no `checksum.sha256`, matching how the
other externals in that file track their upstreams. A digest on a GitHub
auto-generated archive fails spuriously — GitHub does not guarantee those
archives are byte-stable, and a `git archive` gzip change on 2023-01-30
invalidated such digests ecosystem-wide with no content change at all.

## Known upstream issues

### `mcp.json` exists only because there is no user-scope MCP config file

**Tracking:** [anthropics/claude-code#32145](https://github.com/anthropics/claude-code/issues/32145)
— "[FEATURE] Support MCP server configuration in `~/.claude/settings.json`
(user-managed file)". Opened 2026-03-08, `area:mcp`, still open as of
2026-08-26.

Claude Code has three MCP scopes. Local and user scope both live in
`~/.claude.json`, a file Claude Code writes for itself that also holds the OAuth
session, per-project trust decisions and ~40 machine-managed keys — so it cannot
be version-controlled. Project scope lives in `.mcp.json` at a repository root,
which does not apply to user-global servers. There is no user-scope MCP file.

`~/.claude/mcp.json` is therefore **not** a path Claude Code knows about. It is
read only because the `claude` wrapper in `xnoto/dotfiles`
(`private_dot_shellenv.tmpl`) passes it explicitly:

```sh
claude --mcp-config "${HOME}/.claude/mcp.json" --strict-mcp-config "$@"
```

Verified against the Claude Code docs for CLI 2.1.268 on 2026-09-11:
`--mcp-config` and `--strict-mcp-config` are current, documented and actively
developed. This is a supported mechanism, not a deprecated one. The commonly
cited alternative — `jq`-splicing a tracked fragment into `~/.claude.json`
before each invocation — mutates runtime state and breaks on schema changes;
`--mcp-config` does neither.

**Follow up when #32145 ships.** At that point:

- move the `mcpServers` object from `mcp.json` into `settings.json`
- delete `mcp.json` and drop it from the archive
- remove the `claude` wrapper function from
  `xnoto/dotfiles:private_dot_shellenv.tmpl`, since the flags stop being needed

### Operational consequence: `claude mcp add` is inert here

`--strict-mcp-config` makes Claude Code use *only* the servers passed via
`--mcp-config`. Anything added with `claude mcp add` — at any scope — is
silently ignored, as is any project `.mcp.json`. Add servers by editing
`mcp.json` in this repository and redeploying.

As of 2026-09-11 the top-level `mcpServers` key in `~/.claude.json` is empty and
no project defines local-scope servers, so nothing is currently being masked.
