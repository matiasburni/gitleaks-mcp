# gitleaks-mcp

An [MCP](https://modelcontextprotocol.io) server that lets an AI agent scan code, staged changes and text snippets for hardcoded secrets using [gitleaks](https://gitleaks.io).

**Secrets never reach the model.** Every scan runs with `--redact`, and the `Match`/`Secret` fields are dropped before results are returned. The agent only sees rule, file, line, commit and fingerprint.

> **Status of the engine:** gitleaks is feature-complete and in security-patch-only mode; its author is now building [Betterleaks](https://github.com/betterleaks/betterleaks). The scanner sits behind a small `Engine` interface (`src/gitleaks.ts`) so a Betterleaks backend can be added once v2 is stable without changing the tool contract.

## Tools

| Tool | What it does |
|---|---|
| `scan_staged` | Scan only what is staged (what `git commit` would record). |
| `scan_repo` | Scan a repo's history. Optional `log_opts`, e.g. `@{upstream}..HEAD`. |
| `scan_path` | Scan a file or directory without git. |
| `scan_text` | Scan a string (a snippet or config you're about to write or share). |

All tools are read-only and take `max_findings` (default 50); the total count is always reported.

Also provided: resource `gitleaks://about` (engine version and guarantees) and prompt `pre-push-check`.

## Requirements

- Node >= 20
- `gitleaks` on your PATH (`brew install gitleaks`), or set `GITLEAKS_BIN`

## Use it

```bash
git clone https://github.com/matiasburni/gitleaks-mcp && cd gitleaks-mcp
npm install && npm run build
claude mcp add gitleaks -- node "$(pwd)/dist/src/index.js"
```

Then ask: *"scan my staged changes for secrets"* or run the `pre-push-check` prompt.

Try it interactively with the MCP Inspector: `npm run inspect`.

## How it works (MCP in 60 seconds)

- **Transport:** stdio. The client launches `node dist/src/index.js` and exchanges JSON-RPC over stdin/stdout, so the server never writes logs to stdout (only stderr).
- **Tools** are actions the model calls, declared with a zod input schema and `readOnlyHint` annotations (`src/server.ts`).
- **Resources** are read-only data the client can attach; **prompts** are reusable templates the user invokes.
- **Safety:** no shell (`execFile` with an args array), redaction forced on, secret fields dropped, temp report deleted after each scan.

## Develop

```bash
npm test   # builds, then runs MCP-client end-to-end tests against real gitleaks
```

Test secrets are assembled at runtime so no secret-shaped literal is committed.

## License

MIT - see [LICENSE](LICENSE). Permissive on purpose, so you can reuse, fork and learn from it.

## Security

See [SECURITY.md](SECURITY.md) for the threat model, hardening and known limits (notably: only scan repositories you trust).
