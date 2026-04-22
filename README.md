# cc-csharp-plugin

C# language server plugin for Claude Code using [csharp-ls](https://github.com/razzmatazz/csharp-language-server) — a Roslyn-based LSP.

## How it works

Two-layer LSP setup (same design as [cc-ty-plugin](https://github.com/holo-q/cc-ty-plugin)):

1. **`lspServers`** — registers csharp-ls with Claude Code's native integration so push diagnostics surface automatically as `<new-diagnostics>` reminders after edits.

2. **`mcpServers` (cc-lsp-now bridge)** — spawns csharp-ls as a subprocess and exposes the full LSP protocol as MCP tools with symbol-name addressing, multi-target batching, provenance headers, and filesystem watching via `workspace/didChangeWatchedFiles`.

3. **PreToolUse hook** — redirects Claude Code's built-in `LSP()` tool to the MCP tools, which cover more methods and return compact, agent-friendly output.

## Prerequisites

Two binaries need to be on `PATH`: `csharp-ls` and `uvx`.

### csharp-ls (the LSP)

```bash
dotnet tool install --global csharp-ls
```

Put `~/.dotnet/tools` on your PATH if it isn't already:

```bash
export PATH="$HOME/.dotnet/tools:$PATH"
```

### uv / uvx (to fetch the bridge)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

The bridge (`cc-lsp-now`) is fetched and cached automatically by `uvx` on first plugin spawn — no manual install.

## Tools provided

All standard LSP operations are exposed as `lsp_*` MCP tools: `lsp_references`, `lsp_hover`, `lsp_definition`, `lsp_workspace_symbols`, `lsp_rename`, `lsp_call_hierarchy_{incoming,outgoing}`, `lsp_diagnostics`, `lsp_document_symbols`, `lsp_completion`, `lsp_code_actions`, `lsp_move_file`, and more. All support symbol-name addressing and multi-target batching.

Plus: push diagnostics via the native `lspServers` integration, surfaced automatically after edits.

## Configuration

The plugin ships csharp-ls as the sole LSP. To add a fallback (e.g. OmniSharp for refactorings csharp-ls doesn't implement), override `LSP_SERVERS` in your user settings:

```
LSP_SERVERS=csharp-ls;OmniSharp -lsp
```

Or swap to Microsoft's official Roslyn Language Server via `LSP_REPLACE`:

```
LSP_REPLACE=csharp-ls=dotnet /path/to/Microsoft.CodeAnalysis.LanguageServer.dll --stdio
```

## More Information

- [csharp-ls](https://github.com/razzmatazz/csharp-language-server)
- [cc-lsp-now](https://github.com/holo-q/cc-lsp-now) — the LSP-to-MCP bridge
- [cc-ty-plugin](https://github.com/holo-q/cc-ty-plugin) — sibling plugin for Python using `ty`
