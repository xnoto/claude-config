# MCP routing

Select by the named target environment. If it is unspecified, ask before querying or changing anything.

**Hatch** resources: use only `aws-staging`, `aws-prod`, `argocd-staging-eks`, `argocd-prod-eks`, and `grafana`. These are separate servers, so their tools are named `mcp__<server>__<tool>` with no extra prefix — Hatch Grafana is `mcp__grafana__query_prometheus`, not `grafana_query_prometheus`.

**Make IT Work Cloud** resources: use the direct `makeitwork-<integration>` servers — `makeitwork-apify`, `makeitwork-argocd`, `makeitwork-aws`, `makeitwork-aws-docs`, `makeitwork-cloudflare`, `makeitwork-context7`, `makeitwork-gcp`, `makeitwork-grafana`, `makeitwork-kubernetes`, `makeitwork-parallel-search`, `makeitwork-playwright`, `makeitwork-slidespeak`, `makeitwork-terraform-docs`, and `makeitwork-twilio-docs`. Each is its own remote server at `https://mcp-<integration>.makeitwork.cloud/mcp`, authenticating with `CF-Access-Client-*` headers referenced from the `CF_ACCESS_CLIENT_ID` and `CF_ACCESS_CLIENT_SECRET` environment variables. Every integration's tools are named `mcp__makeitwork-<integration>__<tool>` — for example `mcp__makeitwork-grafana__query_prometheus`, `mcp__makeitwork-argocd__list_applications`, `mcp__makeitwork-kubernetes__pods_list`. AWS itself is one such server, reached as `mcp__makeitwork-aws__aws___<tool>`; the server is not AWS-specific despite that prefix. `github`, `hero-ssh`, and `codebase-memory` have no external endpoint and stay on internal or client-local transports.

**Environment-neutral** tooling also arrives through the direct servers: `mcp__makeitwork-parallel-search__*` (web), `mcp__makeitwork-context7__*` (library docs), `mcp__makeitwork-aws-docs__*`, `mcp__makeitwork-terraform-docs__*`, `mcp__makeitwork-apify__*`. `opentofu-docs` is a standalone server.

## Tool usage

The load-bearing usage instructions are restated here.

**Web (`mcp__makeitwork-parallel-search__*`)**: reach for `web_search` first for factual, current-information, research, comparison, and troubleshooting questions. Its excerpts are meant to be answered from directly — do not fetch every result. Pass multiple `search_queries` in one call rather than chaining. Use `web_fetch` only when the user names a specific URL, you need exact wording, or the excerpts conflict or are clearly insufficient.

**Library docs (`mcp__makeitwork-context7__*`)**: use for any library, framework, SDK, API, CLI tool, or cloud service question — API syntax, configuration, version migration, setup, library-specific debugging — even for well-known ones, and even when you think you know the answer, because training data lags. Prefer it over web search for library docs. Not for refactoring, business-logic debugging, code review, or general programming concepts.

**AWS docs (`mcp__makeitwork-aws-docs__*`)**: `search_documentation` with specific technical terms, then `read_sections` when the table of contents localizes the answer, otherwise `read_documentation`. Paginate long pages with `start_index`. Fall back to `recommend` when repeated searches come up short. Always cite the documentation URL.

**Grafana (`mcp__makeitwork-grafana__*` and `mcp__grafana__*`)**: timestamps without a timezone offset are interpreted as UTC. Include an offset such as `-05:00`, or use relative syntax like `now-1h`.

**Terraform/OpenTofu docs**: query the registry for current provider and module versions *before* generating configuration, and pin what you generate.

**Apify (`mcp__makeitwork-apify__*`)**: search the store first — a relevant Actor usually exists — prefer Actors with higher usage or ratings, and always check an Actor's input schema before running it.

## Codebase Memory routing

`codebase-memory` is a local, derived code-discovery index served by the MCP gateway. Use it only for repositories below `~/git`, and index each repository explicitly rather than indexing the parent directory. Keep shared graph-artifact persistence disabled so repository source is not modified. Indexing changes local derived state and requires confirmation. Its local index can be stale or incomplete; use GitHub for exact file reads, remote branch heads, repository writes, and freshness-critical claims.
