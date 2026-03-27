<div align="center">

# Ninetrix Tool Hub

**Community tool registry for [Ninetrix](https://github.com/NinetrixAI/Ninetrix) agents.**

Use `hub://` in your `agentfile.yaml` to add tools with zero config.

[![Tools](https://img.shields.io/badge/tools-29-blue)]()
[![License](https://img.shields.io/badge/license-Apache%202.0-green)](./LICENSE)

</div>

---

## How It Works

Each tool is a `TOOL.yaml` manifest that declares everything the CLI needs to install and run it — source type, dependencies, credentials, and companion skills. No shell scripts, no manual setup.

```yaml
# In your agentfile.yaml — that's it
tools:
  - { name: gh, source: hub://gh }
  - { name: slack, source: hub://slack }
  - { name: notion, source: hub://notion }
```

```bash
ninetrix build   # CLI reads TOOL.yaml, auto-installs deps, wires credentials
ninetrix run     # credentials forwarded from your host env at runtime
```

The CLI resolves `hub://` tools at build time:
1. Fetches `registry.json` (contains SHA256 hashes per file)
2. Reads the tool's `TOOL.yaml` manifest
3. Verifies SHA256 integrity
4. Auto-generates Dockerfile install commands from declarative `dependencies`
5. Forwards required credentials from host env into the container at runtime

---

## Available Tools

| Tool | Description | Source Type |
|------|-------------|------------|
| [`agent-browser`](./tools/agent-browser) | Headless browser for web agents | mcp |
| [`brave-search`](./tools/brave-search) | Brave Search API | mcp |
| [`claude-code`](./tools/claude-code) | Claude Code integration | mcp |
| [`cloudflare`](./tools/cloudflare) | Cloudflare Workers + DNS management | mcp |
| [`duckduckgo`](./tools/duckduckgo) | DuckDuckGo web search | mcp |
| [`fetch`](./tools/fetch) | HTTP fetch (GET/POST any URL) | mcp |
| [`filesystem`](./tools/filesystem) | Local filesystem read/write | mcp |
| [`gh`](./tools/gh) | GitHub CLI — repos, issues, PRs, Actions | cli |
| [`github`](./tools/github) | GitHub MCP server | mcp |
| [`godaddy`](./tools/godaddy) | GoDaddy domain management | mcp |
| [`gogcli`](./tools/gogcli) | Google Cloud CLI | cli |
| [`google-drive`](./tools/google-drive) | Google Drive files | mcp |
| [`gws`](./tools/gws) | Google Workspace (Docs, Sheets, etc.) | mcp |
| [`httpbin`](./tools/httpbin) | HTTP testing endpoints | openapi |
| [`hub-runtime`](./tools/hub-runtime) | Hub runtime utilities | mcp |
| [`linear`](./tools/linear) | Linear issue tracking | mcp |
| [`memory`](./tools/memory) | Persistent agent memory | mcp |
| [`notion`](./tools/notion) | Notion pages and databases | mcp |
| [`ocr`](./tools/ocr) | OCR / image text extraction | local |
| [`openai`](./tools/openai) | OpenAI API integration | openapi |
| [`petstore`](./tools/petstore) | Petstore demo (OpenAPI example) | openapi |
| [`postgres`](./tools/postgres) | PostgreSQL database access | mcp |
| [`puppeteer`](./tools/puppeteer) | Puppeteer browser automation | mcp |
| [`sendgrid`](./tools/sendgrid) | SendGrid email delivery | openapi |
| [`slack`](./tools/slack) | Slack messaging | mcp |
| [`sqlite`](./tools/sqlite) | SQLite database access | mcp |
| [`stripe`](./tools/stripe) | Stripe payments API | openapi |
| [`tavily`](./tools/tavily) | Tavily AI search | mcp |
| [`wrangler`](./tools/wrangler) | Cloudflare Wrangler CLI | cli |

---

## TOOL.yaml Format

Every tool is a single `TOOL.yaml` file:

```yaml
name: gh
version: 1.0.0
description: "GitHub CLI — repos, issues, PRs, releases, Actions"
author: ninetrix
tags: [developer, git, github]
verified: true

# How the tool runs
source:
  type: cli                    # cli | mcp | openapi | local
  # For MCP tools:
  # runner: npx
  # package: "@modelcontextprotocol/server-github"
  # For OpenAPI tools:
  # spec_url: https://api.example.com/openapi.json

# What gets installed in the agent container
dependencies:
  pip: [package1, package2]    # Python packages
  apt: [package1]              # System packages
  npm: [package1]              # Node.js packages (auto-installs Node 22)
  apt_repo:                    # Custom APT repositories
    keyring_url: https://...
    repo: "deb [signed-by=...] https://... stable main"

# Required credentials (forwarded from host env at runtime)
credentials:
  GH_TOKEN:
    label: GitHub personal access token
    required: true

# Map common env var names to what the tool expects
credential_aliases:
  GITHUB_TOKEN: GH_TOKEN

# Companion skills that teach the agent HOW to use this tool
skill_set:
  - hub://gh-master@1.0.0
```

### Source Types

| Type | How it runs | Example |
|------|------------|---------|
| `mcp` | MCP server subprocess via npx/uvx | Slack, GitHub, Notion |
| `cli` | System CLI tool installed via apt/npm | `gh`, `wrangler` |
| `openapi` | REST API with OpenAPI 3.x spec | Stripe, SendGrid |
| `local` | Python `@Tool` function bundled in the tool | OCR |

### Dependencies

All dependencies are **declarative** — no shell commands in TOOL.yaml:

- `pip:` — Python packages (`pip install`)
- `apt:` — System packages (`apt-get install`)
- `npm:` — Node.js packages (auto-installs Node.js 22 + packages)
- `apt_repo:` — Custom APT repositories with GPG keyring

### Companion Skills

Tools provide capabilities. Skills teach the agent HOW to use them effectively. This is the **"Oven + Baker"** pattern:

```yaml
# TOOL.yaml — the oven (capabilities)
skill_set:
  - hub://gh-master@1.0.0
```

```markdown
<!-- SKILL.md — the baker (knowledge) -->
requires:
  tools: [hub://gh]
```

When you `ninetrix hub add gh`, the CLI suggests installing companion skills automatically.

---

## CLI Commands

```bash
# Search tools and skills
ninetrix hub search "github"

# Add a tool to your agentfile.yaml (suggests companion skills)
ninetrix hub add gh

# List all available tools
ninetrix tools list

# Show full details (deps, credentials, skills)
ninetrix tools info gh

# Add tool to agentfile.yaml
ninetrix tools add gh

# Supply chain inspection (SHA256, static analysis)
ninetrix tools inspect gh
```

---

## Supply Chain Security

- **SHA256 verification** — every TOOL.yaml is hashed in `registry.json`; CLI verifies before install
- **Static analysis** — CI scans for dangerous code patterns
- **`ninetrix tools inspect`** — review hash, dependencies, and credential requirements before adding
- **Consent prompt** — CLI asks before installing new tools with system dependencies
- **No secrets in manifests** — credentials are forwarded from host env at runtime, never baked into images

---

## Contributing a Tool

1. Create a folder in `tools/` with your tool name
2. Add a `TOOL.yaml` following the format above
3. (Optional) Add a companion skill in the [Skills Hub](https://github.com/NinetrixAI/skills-hub)
4. Open a PR — CI validates the manifest and updates `registry.json`

### Guidelines

- Tool names should be short and lowercase: `gh`, `slack`, `postgres`
- Use the simplest source type that works (prefer `cli` over `mcp` if a CLI exists)
- Always declare credentials explicitly — no hidden env var dependencies
- Add `verified: true` only for tools maintained by the Ninetrix team

---

## License

Apache 2.0

---

<div align="center">
  <a href="https://github.com/NinetrixAI/Ninetrix">Ninetrix CLI</a> · <a href="https://github.com/NinetrixAI/skills-hub">Skills Hub</a> · <a href="https://docs.ninetrix.io">Docs</a> · <a href="https://ninetrix.io">Website</a>
</div>
