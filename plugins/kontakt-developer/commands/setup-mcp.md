---
description: Register Kontakt's MCP server with Claude Code.
argument-hint: [port]
---

Register the Kontakt MCP server so the komplete-script dev loop works.

1. Port: use `$1` if given and a valid integer; otherwise default to `3006`.
2. Run:
   ```
   claude mcp add --transport http --scope user kontakt http://localhost:<port>/mcp
   ```
   Substitute the resolved port.
3. If it reports the server name `kontakt` already exists, ask the user whether to overwrite (`claude mcp add --transport http --scope user kontakt http://localhost:<port>/mcp --force` — check `claude mcp add --help` for the exact overwrite flag if `--force` errors) or leave the existing one.
4. Confirm to the user: server name (`kontakt`), URL used, and that Kontakt must be running with the MCP server enabled.
