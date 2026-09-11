# claude-config

Version-controlled Claude Code configuration. This repository is the canonical
producer for the rendered `~/.claude` target; it is consumed by `xnoto/dotfiles`
as a pinned chezmoi archive external, not cloned or applied directly.

| File | Deployed to | Purpose |
| --- | --- | --- |
| `CLAUDE.md` | `~/.claude/CLAUDE.md` | Global instructions, MCP routing |
| `settings.json` | `~/.claude/settings.json` | User-scope settings, permissions, plugins |
| `mcp.json` | `~/.claude/mcp.json` | MCP servers, loaded via `--mcp-config` |

Repository tooling (`README.md`, `LICENSE`, `.gitignore`,
`.pre-commit-config.yaml`, `.secrets.baseline`) is excluded from the archive and
never lands in `~/.claude`.

`~/.claude` belongs to Claude Code, which keeps unmanaged runtime state there
(`projects/`, `sessions/`, `plugins/`, `history.jsonl`). This repository
contributes files to that directory; it does not own it. See the `[".claude"]`
external in `xnoto/dotfiles` for the constraints that follow from that.

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
