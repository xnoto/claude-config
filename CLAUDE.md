# Skills and execution boundaries

- Load `context7` when library or framework documentation is needed; do not load it for unrelated work. Use Claude's native Skill tool, or read the installed `skills/context7/SKILL.md` relative to this file if discovery is unavailable.
- Load `context-mode-routing-policy` before using context-mode to process large output, analyze files, or index public web content. The same relative-file fallback applies. If the policy cannot be loaded, use the dedicated task tools instead of context-mode execution. This policy supplements, rather than shadows, the existing plugin's `context-mode` skill.
- Skills are instructions, not tool installation or permission grants. Use only tools present in the current registry. If a required MCP is unavailable, disclose the limitation and use an approved read-only alternative; do not silently install packages, change configuration, or bypass a denied tool.
- Context-mode is an output-management tool, not a security sandbox. Its execution tools are for bounded read-only local inspection only, never mutations, authenticated API calls, secret retrieval, uploads, or work that requires approval through another tool. Prefer the dedicated MCP for repository, cloud, cluster, and documentation operations.
- Never automatically run shell commands returned by doctor/upgrade tools. Installation, upgrades, configuration changes, and service actions require explicit approval of the exact operation and target.

# MCP routing

Select by the named target environment. If it is unspecified, ask before querying or changing anything.

**Hatch** resources: use only `aws-staging`, `aws-prod`, `kubernetes-staging-eks`, `kubernetes-prod-eks`, `argocd-staging-eks`, `argocd-prod-eks`, and `grafana`. `kubernetes-staging-eks` and `kubernetes-prod-eks` are fixed-context Kubernetes gateway entries for their named environments, distinct from `makeitwork-kubernetes`. These are separate servers, so Claude tools are named `mcp__<server>__<tool>` with no extra prefix — Hatch Grafana is `mcp__grafana__query_prometheus`, not `grafana_query_prometheus`.

**Make IT Work Cloud** resources: use the direct Make IT Work Cloud remote servers. Seven use bare integration names — `apify`, `aws-docs`, `context7`, `parallel-search`, `playwright`, `slidespeak`, and `terraform-docs`. Six use the `makeitwork-` prefix: `makeitwork-argocd`, `makeitwork-aws`, `makeitwork-grafana`, and `makeitwork-kubernetes` retain it because their bare names collide with client integrations that represent other environments; `makeitwork-cloudflare` and `makeitwork-gcp` use it to make the Make IT Work Cloud scope explicit. These names target only Make IT Work Cloud resources, not other environments; the prefix is a naming distinction only, not a security boundary. All thirteen are the same kind of remote server at `https://mcp-<integration>.makeitwork.cloud/mcp`, authenticating with `CF-Access-Client-*` headers referenced from the `CF_ACCESS_CLIENT_ID` and `CF_ACCESS_CLIENT_SECRET` environment variables. Every integration's tools are named `mcp__<server>__<tool>` — for example `mcp__makeitwork-grafana__query_prometheus`, `mcp__makeitwork-argocd__list_applications`, `mcp__makeitwork-kubernetes__pods_list`, `mcp__aws-docs__search_documentation`. AWS itself is one such server, reached as `mcp__makeitwork-aws__aws___<tool>`; the server is not AWS-specific despite that prefix. `github` is a workstation-local loopback gateway entry at `http://127.0.0.1:8767/mcp`; the gateway alone sources the unexported `GITHUB_MCP_TOKEN` credential, so clients carry no token headers and spawn no subprocess. `hero-ssh` and `codebase-memory` have no external endpoint and stay on internal or client-local transports; `codebase-memory` remains the local derived index covered in its own section below.

`makeitwork-cloudflare` maps to `https://mcp-cloudflare.makeitwork.cloud/mcp`; `makeitwork-gcp` maps to `https://mcp-gcp.makeitwork.cloud/mcp`. The `makeitwork-` prefix scopes the client-side name only; do not prepend it to the endpoint hostname.

**Environment-neutral** tooling also arrives through the direct servers: `mcp__parallel-search__*` (web), `mcp__context7__*` (library docs), `mcp__aws-docs__*`, `mcp__terraform-docs__*`, and `mcp__apify__*`. `opentofu-docs` is a standalone server.

## Tool usage

The load-bearing usage instructions are restated here.

**Web (`mcp__parallel-search__*`)**: reach for `web_search` first for factual, current-information, research, comparison, and troubleshooting questions. Its excerpts are meant to be answered from directly — do not fetch every result. Pass multiple `search_queries` in one call rather than chaining. Use `web_fetch` only when the user names a specific URL, you need exact wording, or the excerpts conflict or are clearly insufficient. Use context-mode fetch/index only when a known public page needs substantial processing; do not refetch sufficient excerpts merely to index them.

**Library docs (`mcp__context7__*`)**: use for library, framework, SDK, API, and CLI documentation — syntax, configuration, version migration, setup, and library-specific debugging — even for well-known libraries or questions phrased as "latest". Prefer the dedicated `aws-docs`, `terraform-docs`, and `opentofu-docs` integrations for their domains. For OpenCode configuration, use its schema and canonical repository guidance; do not assume a retired OpenCode docs MCP exists. Use Context7 before general web search for library docs, but not for refactoring, business-logic debugging, code review, or general programming concepts.

**AWS docs (`mcp__aws-docs__*`)**: `search_documentation` with specific technical terms, then `read_sections` when the table of contents localizes the answer, otherwise `read_documentation`. Paginate long pages with `start_index`. Fall back to `recommend` when repeated searches come up short. Always cite the documentation URL.

**Grafana (`mcp__makeitwork-grafana__*` and `mcp__grafana__*`)**: timestamps without a timezone offset are interpreted as UTC. Include an offset such as `-05:00`, or use relative syntax like `now-1h`.

**Terraform/OpenTofu docs**: query the registry for current provider and module versions *before* generating configuration, and pin what you generate.

**Apify (`mcp__apify__*`)**: search the store first — a relevant Actor usually exists — prefer Actors with higher usage or ratings, and always check an Actor's input schema before running it.

## Codebase Memory routing

`codebase-memory` is a local, derived code-discovery index served by the MCP gateway. Use it only for repositories below `~/git`, and index each repository explicitly rather than indexing the parent directory. Keep shared graph-artifact persistence disabled so repository source is not modified. Indexing changes local derived state and requires confirmation. Its local index can be stale or incomplete; use GitHub for exact file reads, remote branch heads, repository writes, and freshness-critical claims.

## Bounded specialist delegation

Use the following supplied-material subagents selectively; do not run the whole
roster for every change:

- `adversarial-code-reviewer`: independent review of a completed non-trivial
  implementation before opening its PR, or an explicitly requested second opinion.
- `qa-engineer`: acceptance-to-check coverage, concrete test cases, and
  documentation adequacy; it does not implement tests or execute checks.
- `docs-writer`: standalone README, guide, and release-note drafting from
  supplied sources; not agent instructions, skills, policies, or knowledge bases.
- `infra-security-reviewer`: infrastructure-diff security review of privileges,
  secret handling, exposure, and supply-chain boundaries; never secret values.
- `devops-engineer`: DESIGN or CHANGE review of CI, workflow, artifact, runner,
  and delivery integration contracts; not implementation or live operations.
- `release-engineer`: versions, pins, generated copies, release documentation,
  and downstream stages against the actual repository release contract.
- `cloud-architecture-reviewer`: before implementing a new cloud service or
  material topology, state, recovery, scaling, service-selection, or cost change.

The primary gathers and supplies the complete relevant evidence, acceptance
criteria, repository guidance, producer-consumer context, and validation status.
These seven agents do not retrieve sources or load skills: their supplied-material
contract takes precedence over task-related tool/skill routing in this file.
Keep universal safety constraints, and never supply prohibited secret or state
material. Use fresh reviewer context, not a full-history fork of the authoring
session. Missing essential evidence must produce HOLD or BLOCKED, not invented
facts. Resolve Critical/High findings or obtain an explicit owner waiver; do not
treat a verdict as merge, publication, deployment, or runtime authorization.
The primary retains implementation and final decisions.

If native agent discovery is unavailable, report that limitation; do not pretend
a named independent review occurred or silently install/repair the client.
Models are inherited unless explicitly selected by the caller. Do not substitute
provider aliases for specialist roles.
