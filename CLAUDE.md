# MCP routing

Select by the named target environment. If it is unspecified, ask before querying or changing anything.

**Hatch** resources: use only `aws-staging`, `aws-prod`, `argocd-staging-eks`, `argocd-prod-eks`, and `grafana`. These are separate servers, so their tools are named `mcp__<server>__<tool>` with no extra prefix — Hatch Grafana is `mcp__grafana__query_prometheus`, not `grafana_query_prometheus`.

**Make IT Work Cloud** resources: use only `makeitwork-aws`. It is an aggregating gateway, so every integration reaches you through it as `mcp__makeitwork-aws__<integration>_<tool>` — for example `mcp__makeitwork-aws__makeitwork-grafana_query_prometheus`, `mcp__makeitwork-aws__makeitwork-argocd_list_applications`, `mcp__makeitwork-aws__makeitwork-kubernetes_pods_list`. There are no standalone `makeitwork-argocd`, `makeitwork-kubernetes`, or `makeitwork-grafana` servers.

**Environment-neutral** tooling also arrives through the `makeitwork-aws` gateway: `makeitwork-parallel-search_*` (web), `makeitwork-context7_*` (library docs), `makeitwork-aws-docs_*`, `makeitwork-terraform-docs_*`, `makeitwork-apify_*`. `opentofu-docs` is a standalone server.

## Tool usage

The aggregating gateway does not forward its integrations' own usage instructions, so the load-bearing ones are restated here.

**Web (`makeitwork-parallel-search_*`)**: reach for `web_search` first for factual, current-information, research, comparison, and troubleshooting questions. Its excerpts are meant to be answered from directly — do not fetch every result. Pass multiple `search_queries` in one call rather than chaining. Use `web_fetch` only when the user names a specific URL, you need exact wording, or the excerpts conflict or are clearly insufficient.

**Library docs (`makeitwork-context7_*`)**: use for any library, framework, SDK, API, CLI tool, or cloud service question — API syntax, configuration, version migration, setup, library-specific debugging — even for well-known ones, and even when you think you know the answer, because training data lags. Prefer it over web search for library docs. Not for refactoring, business-logic debugging, code review, or general programming concepts.

**AWS docs (`makeitwork-aws-docs_*`)**: `search_documentation` with specific technical terms, then `read_sections` when the table of contents localizes the answer, otherwise `read_documentation`. Paginate long pages with `start_index`. Fall back to `recommend` when repeated searches come up short. Always cite the documentation URL.

**Grafana (`makeitwork-grafana_*` and `mcp__grafana__*`)**: timestamps without a timezone offset are interpreted as UTC. Include an offset such as `-05:00`, or use relative syntax like `now-1h`.

**Terraform/OpenTofu docs**: query the registry for current provider and module versions *before* generating configuration, and pin what you generate.

**Apify (`makeitwork-apify_*`)**: search the store first — a relevant Actor usually exists — prefer Actors with higher usage or ratings, and always check an Actor's input schema before running it.

## Codebase Memory routing

`codebase-memory` is a local, derived code-discovery index served by the MCP gateway. Use it only for repositories below `~/git`, and index each repository explicitly rather than indexing the parent directory. Keep shared graph-artifact persistence disabled so repository source is not modified. Indexing changes local derived state and requires confirmation. Its local index can be stale or incomplete; use GitHub for exact file reads, remote branch heads, repository writes, and freshness-critical claims.
