# MCP routing

For Hatch resources, use only `aws-staging`, `aws-prod`, `argocd-staging-eks`, `argocd-prod-eks`, and `grafana` (`grafana_*`). For Make IT Work Cloud resources, use only `makeitwork-aws`, `makeitwork-argocd`, `makeitwork-kubernetes`, and `makeitwork-grafana` (`makeitwork-grafana_*`). `apify`, `aws-docs`, `context7`, `parallel-search`, and `terraform-docs` are environment-neutral. Select by the named target environment; if it is unspecified, ask before querying or changing anything.

## Codebase Memory routing

`codebase-memory` is a local, derived code-discovery index served by the MCP gateway. Use it only for repositories below `~/git`, and index each repository explicitly rather than indexing the parent directory. Keep shared graph-artifact persistence disabled so repository source is not modified. Its local index can be stale or incomplete; use GitHub for exact file reads, remote branch heads, repository writes, and freshness-critical claims.
