# Contributing to gemara-ai

## lola Module Details

### Marketplace registration

Register the Gemara marketplace so the module is discoverable via `lola search`:

```bash
lola market add gemara https://raw.githubusercontent.com/gemaraproj/gemara-ai/main/lola-market.yml
lola search gemara
lola mod add https://github.com/gemaraproj/gemara-ai.git
lola install gemara-ai -a <assistant>
```

As of lola 0.7.1, `lola mod add` takes a source (git URL, archive, or folder) — the
marketplace provides discovery, and the `repository` field of the catalog entry is the
URL to pass to `mod add`.

### Reproducible project setup

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

### Verifying the module

Verify the module before publishing a change:

```bash
lola mod rm gemara-ai
lola mod add .
lola mod info gemara-ai
```

Both `gemara-mcp` and `gemara-mcp-advisory` should be listed under MCPs. Assistants without
MCP support (Gemini CLI, Copilot) receive the skills and instructions only.
