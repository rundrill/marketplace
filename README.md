# RunDrill plugin marketplace

The public **install index** for RunDrill courses. Each course ships as one plugin — a
`<subject>-coach` skill backed by an MCP drill engine (curriculum, progress, mistake memory).

This repo holds only the index (`.claude-plugin/marketplace.json`). The course plugins themselves
live in their own repos under the [`rundrill`](https://github.com/rundrill) org and are referenced
from the index by `github` source. Authoring scaffolding (`_template/`) lives in the private
development monorepo, not here.

> No courses published yet — `plugins` in the index is empty. Entries are added as courses ship.

## Installing (for users)

**Claude Code / Claude Desktop** — via this marketplace:
```
/plugin marketplace add rundrill/marketplace        # the org/repo of this index
/plugin install rundrill-english@rundrill            # <plugin-name>@<marketplace-name>
/reload-plugins                                      # → /english-coach is live
```
`@rundrill` is the marketplace `name` field (set in `marketplace.json`), not the repo name.

**Google Antigravity** — no marketplace; Antigravity scans plugin dirs. Drop a course folder in:
- `~/.gemini/config/plugins/rundrill-english/` (global, all workspaces), or
- `<workspace>/.agents/plugins/rundrill-english/` (workspace-scoped).

## Registering a published course

A course plugin lives in its own repo (e.g. `rundrill/rundrill-english`). Add it to the index by
appending a `github`-source entry to `.claude-plugin/marketplace.json` — pin a released tag with
`ref` (and optionally a full-commit `sha`) so users don't pull unfinished work:

```json
{
  "name": "rundrill-english",
  "source": { "source": "github", "repo": "rundrill/rundrill-english", "ref": "v0.1.0" },
  "description": "English coach — drills, progress, and mistake memory."
}
```

## Naming convention

| Layer | Pattern | Example | Notes |
|---|---|---|---|
| Plugin (install unit) | `rundrill-<subject>` | `rundrill-english` | unique within the marketplace; the Antigravity plugin dir name too |
| Skill / slash command | `<subject>-coach` | `/english-coach` | subject-distinct so the bare command works; Claude falls back to `/rundrill-english:english-coach` on a clash |
| MCP server | `rundrill-<subject>` | server key in both `.mcp.json` and `mcp_config.json` | per-course name; one shared backend behind per-course endpoints |

Rules: lowercase **kebab-case** everywhere (not underscores); hyphenate multi-word subjects
(`system-design`, `soft-skills`); lowercase acronyms (`toefl`, `ielts`, `owasp`). The SKILL.md
`name:` field, its folder name, and the slash command must all match.

Planned regional bundles use a distinct word so they never collide with a single course:
`rundrill-pack-west`, `rundrill-pack-asia` (thin meta-plugins listing course plugins as deps).

## Notes

- A plugin folder must be **self-contained** — its skills and MCP config can't reference files
  outside the plugin, because users install one plugin at a time. Shared harness logic lives
  server-side in the MCP backend, not in the plugin.
- **Remote MCP shapes differ:** Claude uses `{ "type": "http", "url": … }` in `.mcp.json`;
  Antigravity uses `{ "serverUrl": … }` in `mcp_config.json`. If RunDrill's OAuth server supports
  **dynamic client registration (DCR)**, Antigravity needs only `serverUrl` — it handles the OAuth
  flow itself. Otherwise add `oauth: { clientId, clientSecret }` and register
  `https://antigravity.google/oauth-callback` as a redirect URI.

Authoring a new course and the submodule layout: see the private monorepo's `plugins/README.md`
and [Plugin Packaging & Distribution](https://github.com/kmmbvnr/rundrill/blob/main/wiki/plugin-packaging.md).
