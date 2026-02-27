# CLAUDE.md — Provenance Toolkit Hub

## Purpose

This is the umbrella/hub repo for the Datafund Provenance Toolkit. It contains no application code — only a developer-facing README that overviews the toolkit and links to individual component repos.

## Sub-Repos

| Component | Local Path | GitHub |
|-----------|-----------|--------|
| SDK | `../swarm_provenance_SDK/` | [datafund/swarm_provenance_SDK](https://github.com/datafund/swarm_provenance_SDK) |
| CLI | `../swarm_provenance_CLI/` | [datafund/swarm_provenance_CLI](https://github.com/datafund/swarm_provenance_CLI) |
| MCP Server | `../swarm_provenance_mcp/` | [datafund/swarm_provenance_mcp](https://github.com/datafund/swarm_provenance_mcp) |
| Gateway | `../swarm_connect/` | [datafund/swarm_connect](https://github.com/datafund/swarm_connect) |
| Landing Page | `../provenance-landing/` | [datafund/provenance-landing](https://github.com/datafund/provenance-landing) |

## Keeping the README Up to Date

Periodically check each sub-repo for changes and update `README.md` accordingly:

1. **Read each sub-repo's key files:**
   - `README.md` — feature list, install commands, usage examples
   - `CLAUDE.md` — architecture, tech stack changes
   - `package.json` or `pyproject.toml` — version bumps, new dependencies

2. **What to look for:**
   - New features or removed features
   - Version changes (update if significant)
   - New or changed install commands
   - API changes that affect quick-start examples
   - New components added to the ecosystem

3. **What to update in README.md:**
   - Components table (descriptions, new repos)
   - Key features list
   - Quick-start code snippets
   - Architecture diagram (if data flow changes)
   - Status section (alpha → beta → stable)

## Conventions

- Keep the README concise and developer-oriented — detailed docs live in individual repos
- Do not duplicate API reference or configuration docs here
- Link to individual repos for anything beyond a quick-start snippet
- Authorship should remain neutral — no attribution to specific tools or agents
