# Idun Blue for Claude Code and Codex

Installs the Idun Blue operating manual as a skill and connects the live MCP
server (https://api.idun.blue/mcp). Generated from the server; do not edit by hand — run
`bun scripts/generate-agent-plugin.ts --write` in the idun-blue repo.

## Install

Claude Code:

```
claude plugin marketplace add idunblue/idun-blue-plugin
claude plugin install idun-blue@idun-blue
claude mcp login idun-blue
```

Codex:

```
codex plugin marketplace add idunblue/idun-blue-plugin
codex plugin add idun-blue@idun-blue
```

Codex asks for the Idun OAuth login on install (`authentication: ON_INSTALL`).

The last line opens the creator’s browser for Idun OAuth. Nothing about her
subscription is stored by Idun; the plugin holds no key.

Skill version: 4.15.0.
