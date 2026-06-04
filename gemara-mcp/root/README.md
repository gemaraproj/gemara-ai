# gemara-mcp

Gemara MCP Server - A Model Context Protocol server for Gemara artifact management.

## Building

Build the binary:

```bash
make build
```

## Installation

### MCP Client Configuration

To use this server with an MCP client, add it to your MCP configuration file.

Add the following configuration (adjust the path to your binary):

```json
{
  "mcpServers": {
    "gemara-mcp": {
      "command": "/absolute/path/to/gemara-mcp/bin/gemara-mcp",
      "args": ["serve"]
    }
  }
}
```

#### Using Docker

If running from Docker, use:

```json
{
  "mcpServers": {
    "gemara-mcp": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "ghcr.io/gemaraproj/gemara-mcp@sha256:0d05c93d237c08483a2b046cff16b1765c42f3cfcba152b02b0904da7d8a05f0",
        "serve"
      ]
    }
  }
}
```

> **Digest pinning**: The image reference above uses `@sha256:...` instead of a mutable tag like `:0.4.0`. This ensures you always pull the exact image that was signed and released. To find the digest for a future release, run:
> ```bash
> docker manifest inspect ghcr.io/gemaraproj/gemara-mcp:<TAG> | jq -r '.digest // .config.digest'
> ```

## Server Modes

The server operates in one of two modes, selected with the `--mode` flag (default: `artifact`).

| Mode | Purpose |
|:---|:---|
| `advisory` | Read-only analysis and validation of existing artifacts |
| `artifact` | All advisory capabilities plus guided artifact creation wizards |

```bash
gemara-mcp serve --mode advisory
gemara-mcp serve --mode artifact
```

## Available Tools, Resources, and Prompts

### Tools

| Tool | Description |
|:---|:---|
| `validate_gemara_artifact` | Validate YAML content against Gemara CUE schema definitions |
| `migrate_gemara_artifact` | Migrate a Gemara artifact to v1 schema using CUE transformations |

### Resources

| Resource URI | Description |
|:---|:---|
| `gemara://lexicon` | Term definitions for the Gemara security model |
| `gemara://schema/definitions` | CUE schema definitions for all Gemara artifact types (latest version) |
| `gemara://schema/definitions{?version}` | CUE schema definitions for a specific Gemara module version |

### Prompts (artifact mode only)

| Prompt | Description |
|:---|:---|
| `threat_assessment` | Interactive wizard for creating a Gemara-compatible Threat Catalog |
| `control_catalog` | Interactive wizard for creating a Gemara-compatible Control Catalog |
| `migration` | Interactive wizard that guides you through migrating Gemara artifacts from v0 to v1 schema |


## Testing the Configuration

Pull the image and verify the MCP server starts and responds:

```bash
docker run --rm -i \
  ghcr.io/gemaraproj/gemara-mcp@sha256:0d05c93d237c08483a2b046cff16b1765c42f3cfcba152b02b0904da7d8a05f0 \
  serve --help
```

Send an MCP `initialize` handshake over stdin to confirm the server speaks the protocol:

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"0.1.0"}}}' \
  | docker run --rm -i \
    ghcr.io/gemaraproj/gemara-mcp@sha256:0d05c93d237c08483a2b046cff16b1765c42f3cfcba152b02b0904da7d8a05f0 \
    serve
```

A successful response returns JSON with `"method":"initialize"` result containing the server's capabilities and version.

## Verifying Image Signatures

Released container images are signed with [cosign](https://docs.sigstore.dev/cosign/overview/) using keyless signing via GitHub Actions OIDC.
Signatures are attached to the image manifest digest.

```bash
cosign verify \
  --certificate-identity-regexp="https://github.com/gemaraproj/gemara-mcp/.github/workflows/release.yml" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/gemaraproj/gemara-mcp@sha256:0d05c93d237c08483a2b046cff16b1765c42f3cfcba152b02b0904da7d8a05f0
```

## Building Docker Image

```bash
docker build --build-arg VERSION=$(git describe --tags --always) --build-arg BUILD=$(git rev-parse --short HEAD) -t gemara-mcp .
```
