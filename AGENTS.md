# Gemara artifact authoring

This module provides skills and MCP tools for authoring [Gemara](https://gemara.openssf.org)
security artifacts (threat catalogs, control catalogs, risk catalogs, policies, mapping documents).

## Skills

| Skill | Use for |
|-------|---------|
| `gemara-artifact-authoring` | Triage + Layer 2 artifacts (ThreatCatalog, ControlCatalog) |
| `gemara-risk-catalog` | Layer 3 RiskCatalog |
| `gemara-policy` | Layer 3 Policy |
| `gemara-mapping-document` | MappingDocument between two existing artifacts |

Start with `gemara-artifact-authoring` when the target artifact type is unknown.

## MCP servers

- `gemara-mcp` — `validate_gemara_artifact`, `migrate_gemara_artifact`, plus Gemara lexicon and schema resources.
- `gemara-mcp-advisory` — read-only mode: validation only, no migration or wizard prompts.

Both run the `ghcr.io/gemaraproj/gemara-mcp` container via `podman`. If you use Docker,
replace `podman` with `docker` in the generated MCP config, or symlink
`ln -s $(which docker) /usr/local/bin/podman`.

## Rules

- Never hand-write a Gemara artifact without validating it — run `validate_gemara_artifact`
  against the CUE schema before declaring an artifact complete.
- Respect layer dependencies: Layer 3 artifacts (Policy, RiskCatalog) reference Layer 2
  artifacts (ControlCatalog, ThreatCatalog). Create the dependency first.
- Use the schema resources exposed by the MCP server as the source of truth for field
  names and enums rather than recalling them from memory.
