# gemara-ai

Gemara's Claude Code plugin for security artifact authoring and validation.

## Installation

Install via Claude Code:

```bash
claude plugin install gemara
```

This installs both the **gemara-mcp** MCP server and the **gemara-artifact-authoring** skill.

### Prerequisites

- [Podman](https://podman.io/docs/installation) or [Docker](https://docs.docker.com/get-docker/) (for the gemara-mcp container)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)

> **Note:** The plugin's `.mcp.json` uses `podman` as the container runtime command.
> If you use Docker, either create a symlink (`ln -s $(which docker) /usr/local/bin/podman`)
> or replace `"podman"` with `"docker"` in `.mcp.json`.

### What you get

| Component | Description |
|-----------|-------------|
| `gemara-mcp` server | MCP server providing `validate_gemara_artifact`, `migrate_gemara_artifact` tools and Gemara lexicon/schema resources |
| `gemara-mcp-advisory` server | Read-only mode — validation only, no migration or wizard prompts |
| `gemara-artifact-authoring` skill | Interactive artifact authoring with triage, prerequisite checking, and step-by-step wizards |

## Supported Artifact Types

| Layer | Artifact Type | Wizard |
|-------|--------------|--------|
| 2 | ThreatCatalog | Threat Assessment |
| 2 | ControlCatalog | Control Catalog |
| 3 | RiskCatalog | Risk Catalog |
| 3 | Policy | Policy |
| 3 | MappingDocument | Mapping Document |

## Usage

The skill automatically triages what artifact to build by:

1. Scanning your repo for existing Gemara artifacts
2. Analyzing any content you provide (YAML, docs, risk registers)
3. Recommending the next artifact based on layer dependencies

Every artifact produced is validated against the Gemara CUE schema before completion.

## Installation with lola (agent-agnostic)

[lola](https://github.com/LobsterTrap/lola) is a package manager that installs skills, MCP
servers, and instructions into any supported AI assistant (Claude Code, Cursor, OpenCode,
Gemini CLI, GitHub Copilot, OpenClaw) from a single source.

Install lola, then add the module directly from this repository:

```bash
lola mod add https://github.com/gemaraproj/gemara-ai.git
lola install gemara-ai -a claude-code
```

`-a` takes one assistant per invocation — repeat the `lola install` line for each of
`claude-code`, `copilot-cli`, `copilot-vscode`, `cursor`, `gemini-cli`, `openclaw`,
`opencode`. Omit `-a` to choose interactively. Add `-s user` for a user-scoped install
instead of the current project.

Or register the Gemara marketplace so the module is discoverable via `lola search`:

```bash
lola market add gemara https://raw.githubusercontent.com/gemaraproj/gemara-ai/main/lola-market.yml
lola search gemara
lola mod add https://github.com/gemaraproj/gemara-ai.git
lola install gemara-ai -a <assistant>
```

As of lola 0.7.1, `lola mod add` takes a source (git URL, archive, or folder) — the
marketplace provides discovery, and the `repository` field of the catalog entry is the
URL to pass to `mod add`.

For reproducible project setup, commit a `.lola-req` file:

```
gemara-ai
https://github.com/gemaraproj/gemara-ai.git#assistant=claude-code,cursor,opencode
```

then run `lola sync`.

### Module layout

lola auto-discovers module content from the repository root:

| Path | Purpose |
|------|---------|
| `skills/<name>/SKILL.md` | Skills, installed to each assistant's skill location |
| `mcps.json` | MCP server definitions, merged into each assistant's MCP config |
| `AGENTS.md` | Module instructions, merged into `CLAUDE.md` / `GEMINI.md` / `AGENTS.md` |

`mcps.json` is lola's canonical MCP format; `.mcp.json` is the Claude Code plugin's copy.
Keep the two in sync when changing the container digest or server arguments.

Verify the module before publishing a change:

```bash
lola mod rm gemara-ai
lola mod add .
lola mod info gemara-ai
```

Both `gemara-mcp` and `gemara-mcp-advisory` should be listed under MCPs. Assistants without
MCP support (Gemini CLI, Copilot) receive the skills and instructions only.

## Local Development

### 1. Clone the repository

```bash
git clone https://github.com/gemaraproj/gemara-ai.git
cd gemara-ai
```

### 2. Pull the container image

With Podman:

```bash
podman pull ghcr.io/gemaraproj/gemara-mcp@sha256:b7575c9d230df8f074e3c939b38df17bd46db0f44cb01b308957206b4c97b6e3
```

With Docker:

```bash
docker pull ghcr.io/gemaraproj/gemara-mcp@sha256:b7575c9d230df8f074e3c939b38df17bd46db0f44cb01b308957206b4c97b6e3
```

Optionally, [verify the image signature](#verifying-the-container-image) with cosign before use.

### 3. Validate the plugin

```bash
claude plugin validate .
```

This checks that `plugin.json`, `.mcp.json`, and all skill definitions are well-formed.

### 4. Install the plugin locally

```bash
claude plugin install gemara-ai --scope local
```

### 5. Verify the MCP servers connect

Start a Claude Code session and check that both servers are healthy:

```bash
claude mcp list
claude mcp get gemara-mcp
claude mcp get gemara-mcp-advisory
```

Both servers should show a connected status. If prompted, approve the project-scoped MCP servers from `.mcp.json`.

### 6. Test a skill

From a project directory, start Claude Code and invoke one of the artifact authoring wizards:

```
/gemara:gemara-artifact-authoring
```

The skill will scan the project for existing Gemara artifacts and guide you through creating one.

## MCP Server

The plugin bundles the [gemara-mcp](https://github.com/gemaraproj/gemara-mcp) server (v0.6.0) as a container image. The server provides:

- **Tools:** `validate_gemara_artifact`, `migrate_gemara_artifact`
- **Resources:** `gemara://about`, `gemara://lexicon`, `gemara://schema/definitions`
- **Prompts:** `threat_assessment`, `control_catalog`, `mapping_document`, `policy`, `risk_catalog`, `migration`

### Verifying the container image

```bash
cosign verify \
  --certificate-identity-regexp="https://github.com/gemaraproj/gemara-mcp/.github/workflows/release.yml" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/gemaraproj/gemara-mcp@sha256:b7575c9d230df8f074e3c939b38df17bd46db0f44cb01b308957206b4c97b6e3
```

## License

[Apache License 2.0](LICENSE)
