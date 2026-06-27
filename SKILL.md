---
name: firecrawl-hermes
description: |
  Hermes-native companion to the official Firecrawl CLI skills. Use this skill when calling Firecrawl CLI from Hermes terminal tool on Windows/MSYS. Covers new v1.19.21+ commands not in official skills: research (arXiv + GitHub), doctor, env, make, launch, setup. Adds Windows path pitfalls, credit-saving patterns, and quick command reference. Load alongside firecrawl/scrape/search/crawl/map/parse/agent/interact/monitor skills — this skill is the Hermes-specific layer, those are the command detail.
allowed-tools:
  - terminal
---

# Firecrawl CLI — Hermes Native Companion

Official Firecrawl skills (firecrawl, firecrawl-scrape, firecrawl-search, etc.) are written for Claude Code's `Bash()` tool. This skill bridges them to Hermes `terminal` tool and adds commands they don't cover yet.

## Current Setup

- **CLI version**: v1.19.21+ (check with `firecrawl --version`)
- **Auth**: stored credentials or `FIRECRAWL_API_KEY` env var
- **Credits**: check with `firecrawl --status` or `firecrawl credit-usage`
- **Concurrency**: 5 parallel jobs (check with `firecrawl --status`)
- **Skills**: 31 official skills installed via `firecrawl init --all`
- **Path**: firecrawl CLI available globally via npx

## Hermes Terminal Syntax

Official skills use `Bash(firecrawl ...)`. In Hermes, use the `terminal` tool:

```
terminal(command="firecrawl scrape 'https://example.com' -o .firecrawl/page.md")
```

Always use POSIX shell syntax (this is git-bash/MSYS, not PowerShell):
- Single-quote URLs with `?` or `&`
- Use `$HOME` not `%USERPROFILE%`
- Paths: `/c/Users/...` or `C:\Users\...` both work
- `&&` for chaining, `|` for pipes, `2>&1` for stderr

## Quick Reference — All Commands

| Command | Purpose | Credit Cost |
|---------|---------|-------------|
| `firecrawl scrape <url> -o <path>` | Scrape URL → markdown file | ~1 |
| `firecrawl search <query> -o <path>` | Web search → JSON file | 2 |
| `firecrawl crawl <url> -o <path>` | Crawl entire site section | ~1/page |
| `firecrawl map <url> -o <path>` | List all URLs on a site | 1 |
| `firecrawl parse <file> -o <path>` | Local file → markdown (PDF, DOCX, XLSX) | ~1/page |
| `firecrawl agent <prompt> --urls <u> --wait -o <path>` | AI-powered structured extraction → JSON | variable (use `--model spark-1-mini` to save credits) |
| `firecrawl interact` | Browser interaction (clicks, forms) | variable |
| `firecrawl monitor create ...` | Schedule recurring scrape + change alerts | 1/check |
| `firecrawl research search-papers <query>` | arXiv semantic paper search | ~1 |
| `firecrawl research search-github <query>` | GitHub issue/PR/readme search | ~1 |
| `firecrawl research read-paper <id> -Q <q>` | Read in-body paper passages | ~1 |
| `firecrawl experimental download <url>` | Download site as local files | ~1/page |
| `firecrawl doctor` | Environment diagnostics | 0 |
| `firecrawl env` | Pull API key to .env file | 0 |
| `firecrawl make default` | Set Firecrawl as default provider | 0 |
| `firecrawl launch hermes` | Configure Firecrawl MCP for Hermes | 0 |
| `firecrawl credit-usage` | Check remaining credits | 0 |
| `firecrawl --status` | Version, auth, credits, concurrency | 0 |

## New Commands (v1.19.21+ — not in official skills)

### research — arXiv + GitHub

Semantic search over arXiv abstracts and GitHub issue/PR history. Powerful for literature reviews and finding prior art.

```bash
# Search arXiv papers by topic (returns ranked papers with id, title, abstract)
firecrawl research search-papers "diffusion image synthesis" --limit 20

# Get full metadata for one paper
firecrawl research inspect-paper arxiv:1706.03762

# Expand from seed papers via citation graph (similar, citers, references)
firecrawl research related-papers arxiv:1706.03762 --intent "efficient transformers" --mode similar

# Read specific in-body passages to verify a claim
firecrawl research read-paper arxiv:1706.03762 --question "What is the attention mechanism?"

# Search GitHub issues/PRs/readmes
firecrawl research search-github "foundationdb queue worker shutdown" --limit 10
```

**Tips:**
- Run several distinct query framings instead of one — semantic search rewards variety
- `search-papers` returns up to 40 results by default
- Use `related-papers` after finding strong anchors to discover the paper family
- `read-paper` is for verification, not summarization — use it to check if a paper satisfies a specific constraint

### doctor — Environment Diagnostics

```bash
# Full environment check
firecrawl doctor

# Diagnose a specific failed job
firecrawl doctor <job-id>
firecrawl doctor --run <job-id> --query "why did this run fail?"
```

Checks: CLI version, Node runtime, API key, API reachability, credits, concurrency, .env, .gitignore, AI agent integrations, MCP server. Outputs JSON with `--json`.

### env — Pull API Key

```bash
# Write FIRECRAWL_API_KEY to .env
firecrawl env

# Custom file or overwrite
firecrawl env -f .env.local --overwrite
```

### make — Default Provider

```bash
# Make Firecrawl the default web tool for supported agents
firecrawl make default

# Undo
firecrawl make default --undo
```

### launch — MCP for Agents

Configure Firecrawl MCP for a specific agent, then launch it:

```bash
# Supported agents: claude, code/vscode, codex, codex-app, hermes, openclaw, opencode
firecrawl launch hermes          # Configure MCP for Hermes + launch
firecrawl launch claude          # For Claude Code
firecrawl launch --install       # Install MCP without launching
firecrawl launch --skip-mcp      # Launch without installing/updating MCP
```

### setup — Individual Integrations

```bash
firecrawl setup skills           # Install official skills
firecrawl setup workflows        # Install workflow skills
firecrawl setup mcp              # Install MCP server
firecrawl setup defaults         # Set default provider config
firecrawl setup -y -g            # Global install, skip prompts
```

## Windows/MSYS Pitfalls

1. **URL quoting**: Always single-quote URLs containing `?` or `&` — bash interprets them
   ```bash
   # ✅ Correct
   firecrawl scrape 'https://example.com?page=1&x=2' -o .firecrawl/page.md
   # ❌ Wrong — bash splits on &
   firecrawl scrape https://example.com?page=1&x=2 -o .firecrawl/page.md
   ```

2. **File paths with spaces**: Quote them
   ```bash
   firecrawl parse "./My Document.pdf" -o .firecrawl/mydoc.md
   ```

3. **Output directory**: Always create `.firecrawl/` first and use `-o`
   ```bash
   mkdir -p .firecrawl
   firecrawl scrape 'https://example.com' -o .firecrawl/page.md
   ```
   Without `-o`, output goes to stdout and floods the terminal — can crash long pages.

4. **Parallel jobs**: Use `&` and `wait` in bash
   ```bash
   firecrawl scrape 'https://a.com' -o .firecrawl/a.md &
   firecrawl scrape 'https://b.com' -o .firecrawl/b.md &
   wait
   ```

5. **Don't use PowerShell syntax**: `Get-ChildItem`, `$env:FOO`, `Select-String` won't work. Use `ls`, `$FOO`, `grep`.

6. **`.gitignore`**: Add `.firecrawl/` to avoid committing cached scrape results
   ```bash
   echo ".firecrawl/" >> .gitignore
   ```

7. **`agent` is slow — needs `--wait`**: Agent jobs run async on server, may take 60s+. Use `--wait --timeout 120` or check with `--status` later. Use `--model spark-1-mini` (default) to save credits.
   ```bash
   firecrawl agent "Extract pricing tiers as JSON" --urls 'https://example.com/pricing' --model spark-1-mini --wait --timeout 120 -o .firecrawl/pricing.json
   ```

8. **`download` prompts for confirmation**: Pipe `y` or specify limit directly
   ```bash
   echo "y" | firecrawl experimental download 'https://example.com' --limit 5 --format markdown
   # Or specify limit to skip prompt
   firecrawl experimental download 'https://example.com' --limit 5
   ```

## Credit Check — Always do this before using Firecrawl

```bash
firecrawl credit-usage 2>&1 | head -3
# Or
firecrawl --status 2>&1 | grep Credits
```

- **Plenty left (>200)**: Use freely, no worries
- **Low (<200)**: Conserve — use `web_search`/`web_extract` first, avoid `agent`/`crawl` which cost more credits

## Credit-Saving Patterns

- **`search --scrape`** fetches full content in one call — don't re-scrape those URLs
- **`search-feedback`** refunds 1 credit per search (first feedback only) — always send it after using results
- **Check `.firecrawl/`** before re-scraping the same URL
- **`--only-main-content`** strips nav/footer — cheaper, cleaner output
- **`parse`** costs ~1 credit/page for PDF, 1 flat for HTML — prefer over scrape for local files
- **Monitor** instead of repeated one-off scrapes for ongoing checks

## Workflow Escalation

1. **search** — no URL yet, find pages
2. **scrape** — have a URL, static or JS-rendered
3. **map + scrape** — large site, need to find the right subpage first
4. **crawl** — bulk content from a site section (e.g., all /docs/)
5. **interact** — need clicks, forms, pagination, login
6. **monitor** — recurring checks / change alerts
7. **research** — arXiv papers or GitHub issue history

## When to Use Firecrawl vs Hermes Native Tools

**Benchmark results (tested 2026-06-27):**

| Task | Hermes native | Firecrawl | Winner |
|------|--------------|-----------|--------|
| **Search** | `web_search` — instant, free, rich descriptions | `firecrawl search` — 3s, 2 credits, shorter | **web_search** |
| **Extract** | `web_extract` — ~2s, free, clean formatted markdown | `firecrawl scrape` — 1.8s, 1 credit, raw markdown | **web_extract** |
| **JS SPA** | `web_extract` — handles it | `firecrawl scrape` — handles it | tie |
| **Map** | no equivalent | `firecrawl map` — 1352 URLs in 3.2s, 1 cr | **Firecrawl** |
| **Crawl** | no equivalent | `firecrawl crawl` — 1.9s, 1 cr/pg | **Firecrawl** |
| **Download** | no equivalent | `firecrawl x download` — creates folder structure | **Firecrawl** |
| **Parse** | `read_file` reads but no format conversion | `firecrawl parse` — 1.4s, 1 cr, → markdown | **Firecrawl** |
| **Research** | no equivalent | `firecrawl research` — arXiv + GitHub | **Firecrawl** |
| **Agent** | no equivalent | `firecrawl agent` — slow 60s+, structured JSON | **Firecrawl** (but slow) |
| **Interact** | no equivalent | `firecrawl interact` — browser clicks/forms | **Firecrawl** |
| **Monitor** | no equivalent | `firecrawl monitor` — change alerts + webhook | **Firecrawl** |

**Decision matrix:**

| Need | Use |
|------|-----|
| Web search (general) | `web_search` (primary) → `firecrawl search` (fallback) |
| Web search (news, freshness-sensitive) | `firecrawl search --tbs qdr:d` (primary) |
| Thai-language news | `web_search` (primary) — Firecrawl returns low-quality social posts for Thai content |
| Scrape a URL | `web_extract` (primary) → `firecrawl scrape` (fallback for JS-heavy/SPAs) |
| Crawl a site section | `firecrawl crawl` (Hermes has no equivalent) |
| Map site URLs | `firecrawl map` (Hermes has no equivalent) |
| Monitor page changes | `firecrawl monitor` (Hermes has no equivalent) |
| arXiv paper search | `firecrawl research search-papers` (Hermes has no equivalent) |
| GitHub issue/PR search | `firecrawl research search-github` (Hermes has no equivalent) |
| Parse local PDF/DOCX | `firecrawl parse` (Hermes has no equivalent) |
| Browser interaction | `firecrawl interact` (Hermes has no equivalent) |
| Download site to files | `firecrawl experimental download` (Hermes has no equivalent) |

**Firecrawl wins — no Hermes equivalent:**
- `map` — list all URLs on a site (1352 pages in 3s)
- `crawl` — extract multiple pages following links
- `download` — save site as local folder structure
- `parse` — local PDF/DOCX → markdown (1.4s) — `read_file` reads but doesn't convert formats
- `research` — arXiv semantic search + GitHub issue/PR search
- `interact` — browser clicks/forms/pagination
- `monitor` — schedule recurring scrape + change alerts + webhook
- `agent` — AI structured extraction → JSON (slow 60s+, use `spark-1-mini` to save credits)

**Hermes native wins — use first:**
- `web_search` — search (faster + free + richer descriptions)
- `web_extract` — extract URL (faster + free + cleaner markdown)
- JS-heavy SPA: `web_extract` handles it — use Firecrawl only when `web_extract` fails

**Rule of thumb:** `web_search`/`web_extract` first (free+fast) → Firecrawl when you need crawl/map/monitor/research/parse/interact/download or `web_extract` fails

## Time-Filtered News Search (Critical for Recurring Cron Jobs)

Firecrawl `search` supports `--tbs` (time-based search) — the most important feature for news/freshness-sensitive searches that `web_search` lacks entirely.

```bash
# Past 24 hours
firecrawl search "AI model release news" --tbs qdr:d --limit 5 --json -o .firecrawl/news.json

# Past hour (breaking news)
firecrawl search "AI model release news" --tbs qdr:h --limit 5 --json -o .firecrawl/breaking.json

# Past week
firecrawl search "AI model release news" --tbs qdr:w --limit 5 --json -o .firecrawl/week.json
```

`--tbs` values: `qdr:h` (hour), `qdr:d` (day), `qdr:w` (week), `qdr:m` (month), `qdr:y` (year)

### Why web_search Fails for Recurring News (Stale Reference Page Problem)

Tested with 5 news cron jobs. `web_search` consistently returns **evergreen reference pages** instead of news articles:

| Query | web_search returns | Firecrawl `--tbs qdr:d` returns |
|-------|---------------------|----------------------------------|
| "AI news today" | YouTube recap "AI trends 2025", Microsoft blog | OpenAI GPT-5.6 preview, Anthropic Mythos access |
| "stock market today" | Yahoo Finance quote page, Trading Economics | moneyandbanking.co.th SET close report today |
| "tech framework release" | dev.to tag page, 2024 roundup | Apple WWDC26, .NET out-of-band patch |
| "global economy news" | Bloomberg/MarketWatch overview | Nasdaq 5-day losing streak, Iran drone strike |

The reference pages (Yahoo Finance, Trading Economics, Bloomberg) have **static content that never changes** — so recurring cron jobs that use `web_search` report the same "news" for 3+ consecutive days.

### Hybrid Pattern for News Crons (Thai + International)

For cron jobs covering both Thai and international news, use a hybrid approach:

- **Thai-language topics** (entertainment, local news): `web_search` with Thai query — Firecrawl returns low-quality Instagram/TikTok/Facebook posts for Thai content
- **English/international topics** (AI, tech, markets, world news): `firecrawl search --tbs qdr:d` — consistently fresher, real news articles
- **K-entertainment / Hollywood**: `firecrawl search --tbs qdr:d` — better than web_search for international entertainment

### Cron Job Prompt Pattern

When writing cron job prompts that use Firecrawl for news, include explicit terminal commands in the prompt so the cron agent doesn't improvise:

```
mkdir -p .firecrawl
firecrawl search "AI model release new launch" --tbs qdr:d --limit 5 --json -o .firecrawl/ai-models.json
firecrawl search "AI product app tool update" --tbs qdr:d --limit 5 --json -o .firecrawl/ai-products.json
# Read: jq -r '.data.web[] | "\(.title)\n\(.url)\n\(.description)\n---"' .firecrawl/ai-models.json
```

Attach `firecrawl` skill to the cron job so it loads the CLI reference. Keep `web` toolset enabled too for arxiv/research fallback.

## When to Use This vs Official Skills

- **This skill**: Hermes terminal syntax, new commands, Windows pitfalls, quick reference, tool routing
- **Official `firecrawl`**: Full CLI reference, workflow patterns, monitor JSON config, feedback details
- **Official `firecrawl-scrape/search/crawl/map/parse`**: Deep dive on each command's options
- **Official `firecrawl-research-papers`**: Literature review workflow (uses MCP syntax — adapt to CLI)
- **Official `firecrawl-deep-research`**: Full cited analytical report workflow
- **Official `firecrawl-interact`**: Browser interaction details

Load the relevant official skill for command-level detail. Use this skill for Hermes-specific execution patterns and tool routing.

## References

- [references/benchmark-results.md](references/benchmark-results.md) — Full benchmark data from 2026-06-27 testing: search, extract, JS SPA, map, crawl, parse, download, agent, research, doctor. Includes speed, cost, output size, and pitfalls discovered during real testing.