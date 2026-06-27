# Firecrawl Hermes CLI

Hermes Agent skill for using Firecrawl CLI from Hermes `terminal` tool on Windows/MSYS. Covers all Firecrawl commands including new v1.19.21+ features not in official skills: `research` (arXiv + GitHub), `doctor`, `env`, `make`, `launch`, `setup`. Adds Windows/MSYS pitfalls, credit-saving patterns, benchmark results, and tool routing guidance (when to use Firecrawl vs Hermes native `web_search`/`web_extract`).

## What's included

- `SKILL.md` — Full skill documentation for Hermes Agent
- `references/benchmark-results.md` — Real benchmark data comparing Firecrawl vs Hermes native tools (search, extract, map, crawl, parse, download, agent, research)

## Key Features

- **Quick reference table** for all 18 Firecrawl CLI commands
- **Windows/MSYS pitfalls** — URL quoting, file paths, output handling, PowerShell syntax issues
- **Credit check workflow** — always check credits before using Firecrawl
- **Tool routing** — when to use `web_search`/`web_extract` (free, fast) vs Firecrawl (capabilities Hermes lacks)
- **Time-filtered news search** — `--tbs qdr:d` for freshness-sensitive queries (critical for cron jobs)
- **Benchmark results** — real test data showing where each tool wins

## Quick Start

```bash
# Install Firecrawl CLI
npx -y firecrawl-cli@latest init --all --browser

# Check status
firecrawl --status

# Scrape a page
mkdir -p .firecrawl
firecrawl scrape 'https://example.com' -o .firecrawl/page.md

# Search with time filter (past 24h)
firecrawl search "AI news" --tbs qdr:d --limit 5 --json -o .firecrawl/news.json

# Research arXiv papers
firecrawl research search-papers "LLM agent orchestration" --limit 10

# Download a site
echo "y" | firecrawl experimental download 'https://docs.example.com' --limit 5
```

## Usage with Hermes Agent

Copy `SKILL.md` to your Hermes skills directory:

```bash
cp SKILL.md ~/.hermes/skills/firecrawl-hermes/SKILL.md
```

Or install all official Firecrawl skills + this companion:

```bash
npx -y firecrawl-cli@latest init --all --browser  # installs 31 official skills
# Then add this skill as the Hermes-specific layer
```

## Tool Routing Summary

| Need | Use |
|------|-----|
| Web search (general) | `web_search` (primary) → `firecrawl search` (fallback) |
| Web search (news, freshness) | `firecrawl search --tbs qdr:d` (primary) |
| Scrape a URL | `web_extract` (primary) → `firecrawl scrape` (fallback) |
| Crawl/map/download/monitor | `firecrawl` (Hermes has no equivalent) |
| arXiv/GitHub research | `firecrawl research` (Hermes has no equivalent) |
| Parse local PDF/DOCX | `firecrawl parse` (Hermes has no equivalent) |
| Browser interaction | `firecrawl interact` (Hermes has no equivalent) |

## Requirements

- Node.js 18+ (for npx/firecrawl CLI)
- Firecrawl API key (free tier available, or `firecrawl init --browser` to sign up)
- Hermes Agent (for skill loading) — or any AI agent that can execute terminal commands

## License

MIT — use freely, attribution appreciated.

## Sources

- [Firecrawl CLI official](https://docs.firecrawl.dev/sdks/cli)
- [Firecrawl docs](https://docs.firecrawl.dev)
- [Firecrawl GitHub](https://github.com/firecrawl/firecrawl)