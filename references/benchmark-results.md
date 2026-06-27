# Firecrawl vs Hermes Native — Benchmark Results

> Tested 2026-06-27 on Windows (git-bash/MSYS, CLI v1.19.21)

## Search: `web_search` vs `firecrawl search`

**Query**: "LLM agent orchestration frameworks 2026"

| Metric | `web_search` | `firecrawl search` |
|--------|-------------|-------------------|
| Speed | instant | 3.0s |
| Cost | free | 2 credits |
| Results | 5 results | 5 results |
| Description quality | rich, detailed | short summaries |
| Winner | **web_search** | |

## Extract: `web_extract` vs `firecrawl scrape`

**URL**: `https://docs.firecrawl.dev/features/research`

| Metric | `web_extract` | `firecrawl scrape` |
|--------|-------------|-------------------|
| Speed | ~2s (LLM processing) | 1.8s |
| Cost | free | 1 credit |
| Output size | 5KB+ formatted markdown | 5.4KB raw markdown |
| Readability | clean, structured | raw but complete |
| Winner | **web_extract** (readability) | |

## JS-Heavy SPA: `web_extract` vs `firecrawl scrape`

**URL**: `https://react.dev/reference/react/useEffect`

| Metric | `web_extract` | `firecrawl scrape` |
|--------|-------------|-------------------|
| Result | full content, formatted | full content, raw |
| Speed | ~2s | 3.0s |
| Winner | **tie** — both handle JS SPAs |

## Map (Hermes has no equivalent)

**URL**: `https://docs.firecrawl.dev`

```
firecrawl map 'https://docs.firecrawl.dev' -o .firecrawl/bench-map.json
```

| Metric | Value |
|--------|-------|
| Speed | 3.2s |
| Cost | 1 credit |
| URLs found | 20+ (sample of 1352 total pages) |
| Output format | newline-delimited URLs |

## Crawl (Hermes has no equivalent)

**URL**: `https://docs.firecrawl.dev/introduction`, limit 5

| Metric | Value |
|--------|-------|
| Speed (with --wait) | ~90s (async job) |
| Speed (without --wait) | 1.9s (returns job ID) |
| Cost | 1 credit/page |
| Output | JSON with markdown + metadata per page |

**Pitfall**: Crawl is async. Without `--wait`, returns immediately with a job ID. Use `--wait` to block, or poll with `firecrawl crawl <job-id>`.

## Parse (Hermes `read_file` can't convert formats)

**File**: `.firecrawl/test-doc.html` (local HTML)

| Metric | Value |
|--------|-------|
| Speed | 1.4s |
| Cost | 1 credit (flat for HTML) |
| Output | clean markdown with headings preserved |
| `read_file` alternative | reads raw HTML only, no format conversion |

## Download (Hermes has no equivalent)

**URL**: `https://docs.firecrawl.dev/introduction`, limit 3

```
echo "y" | firecrawl experimental download 'https://docs.firecrawl.dev/introduction' --limit 3 --format markdown
```

| Metric | Value |
|--------|-------|
| Speed | 4.5s (map + scrape 3 pages) |
| Cost | 1 credit/page (3 credits) |
| Output | nested directories under `.firecrawl/` mimicking site structure |
| Files created | `docs.firecrawl.dev/*/index.md` per page |

**Pitfall**: Prompts for confirmation — pipe `y` or specify `--limit` to skip.

## Agent (Hermes has no equivalent)

**URL**: `https://firecrawl.dev/pricing`

```
firecrawl agent "Extract pricing tiers as JSON" --urls 'https://firecrawl.dev/pricing' --model spark-1-mini --wait --timeout 120
```

| Metric | Value |
|--------|-------|
| Speed | 60s+ (timeout at 60s, still processing) |
| Cost | variable (agent credits) |
| Async | yes — job runs on server, poll with `--status` |
| Pitfall | very slow, use `--wait --timeout 120` minimum, or fire-and-forget + poll later |

## Research — arXiv (Hermes has no equivalent)

**Query**: "LLM agent orchestration LLM", limit 3

```
firecrawl research search-papers "LLM agent orchestration LLM" --limit 3
```

| Metric | Value |
|--------|-------|
| Speed | ~3s |
| Cost | ~1 credit |
| Output | ranked papers with arXiv id, title, abstract |

## Research — GitHub (Hermes has no equivalent)

**Query**: "firecrawl CLI v1.19", limit 2

| Metric | Value |
|--------|-------|
| Speed | ~3s |
| Cost | ~1 credit |
| Output | repo name, URL, snippet from issues/PRs/readmes |

## Doctor (free diagnostics)

| Metric | Value |
|--------|-------|
| Speed | ~2s |
| Cost | 0 |
| Checks | CLI version, Node, API key, reachability, credits, concurrency, .env, .gitignore, AI agents, MCP |
| Result | 11 passed, 0 warnings, 0 failed |

## Credit Usage Summary

- Starting credits: ~4,940
- Credits used in testing: ~18 (search 2 + scrape 1 + map 1 + crawl 1 + parse 1 + download 3 + research 2 + agent ~7)
- Remaining: ~4,922