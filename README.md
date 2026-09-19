# Idun Blue for Claude Code and Codex

Installs the Idun Blue operating manual as a skill and connects the live MCP
server (https://api.idun.blue/mcp). The package contains no credentials. Sign in to Idun
through the client's browser-based OAuth flow; never paste keys into a chat.
Generated from the server; maintainers run
`bun scripts/generate-agent-plugin.ts --write` in the idun-blue repo.

## Claude Code: install once

```
claude plugin marketplace add idunblue/idun-blue-plugin
claude plugin install idun-blue@idun-blue
```

Open Claude Code, run `/mcp`, select the Idun Blue plugin server and authenticate.
The plugin server is scoped as `plugin:idun-blue:idun-blue`; it is not the
bare server name used by a separately configured connection. Follow the Idun
login in your browser. The enabled plugin and its skill are available in later
sessions within the installation scope. The CLI defaults to user scope; use
`--scope project` when installation should be declared for this folder.

For updates, open `/plugin` → Marketplaces → idun-blue → Enable auto-update.
Third-party marketplaces have automatic updates off by default. Apply an update
with `/reload-plugins` when prompted, or start a new session. Organization
policies may limit installation or updates.

## Codex: install once

```
codex plugin marketplace add idunblue/idun-blue-plugin
```

Open the Plugins Directory in the desktop app, select the Idun Blue marketplace,
and install Idun Blue. Complete the requested Idun OAuth sign-in. Refresh or
restart the app if the new marketplace is not listed. Marketplace availability
can differ between clients and managed accounts; adding a marketplace alone
does not install the plugin or authorize access.

To refresh the tracked marketplace, run
`codex plugin marketplace upgrade idun-blue`, then reopen the app and check
the installed plugin version. A downloaded local copy does not update itself;
replace it with a fresh package and restart the client to load the new files.

## Keep workspace instructions across tasks

In Studio's own-AI setup, download the workspace starter folder. Open that same
folder as the main project in Codex or Claude Code for future tasks. Keep its
short `AGENTS.md`; `CLAUDE.md` imports it with `@AGENTS.md` for Claude Code.
Preserve existing folder instructions when adding these files to another project.
Plugin installation makes the skill available; it does not inject workspace
instructions into every unrelated task or automatically select a workspace.

Start each new task or recovered conversation with `idun_start`. Follow the
live rules and contracts it returns. Keep the same workspace on OAuth calls
that support the argument; bound key/JWT clients omit it and verify the returned
workspace. Resume observed project/operation IDs before writing again; never
create replacements or replay an uncertain change merely because a chat ended.
The installed manual is a versioned snapshot and still uses context when read.

For Claude or ChatGPT web projects, copy the workspace prompt into the project's
instructions once and connect Idun separately. Uploading a local AGENTS.md file
as reference material is not the same as installing project instructions.

## Package layout

Root `plugin.json`, `mcp.json` and `skills/` form the portable package.
`.codex-plugin/` and `.claude-plugin/` remain for compatible clients.
Use one Idun connection per client; check existing connections before adding a
second MCP configuration for the same URL.

Setup references:
- [OpenAI plugin packaging](https://developers.openai.com/plugins/build/plugins)
- [Claude Code plugins](https://code.claude.com/docs/en/discover-plugins)
- [Claude Code MCP and OAuth](https://code.claude.com/docs/en/mcp)
- [Claude Code project instructions](https://code.claude.com/docs/en/memory)

Skill version: 4.16.0.
