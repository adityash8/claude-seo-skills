# Claude SEO Audit Skill

A comprehensive technical SEO audit skill for Claude — 20 audit sections covering Core Web Vitals, indexability, schema markup, content decay, backlink profile, GEO optimization, CTR gap analysis, and AI surface scoring.

Works with **Ahrefs MCP** (full audit) or **Semrush MCP** (full audit, no per-page crawl). Falls back to web_fetch/web_search if neither is connected.

## What It Does

- **20 audit sections**: Manual actions, redirects, organic traffic, CWV, indexability, robots.txt, sitemaps, canonicalization, content decay, broken links, HTTP issues, schema markup, internal linking, backlink profile, content gap, CTR analysis
- **GEO optimization**: Generates citation-optimized TL;DR blocks for LLM/AI visibility (ChatGPT, Perplexity, Google AIO)
- **AI surface scoring**: Identifies which keywords to optimize for traditional Google vs AI surfaces
- **Stack-aware fix application**: Can auto-apply fixes for Webflow Enterprise, Cloudflare, Vercel, WordPress, or output copy-paste diffs for custom stacks
- **Revenue impact estimates**: Calculates monthly revenue opportunity from traffic recovery

## Prerequisites

At least one of:
- **Ahrefs MCP** — enables full audit including per-page CWV, schema validation, canonical checks, internal linking, and GSC data
- **Semrush MCP** — enables domain-level audit (no per-page crawl data)
- **Neither** — limited audit using web_fetch only

## Setup

### Claude Desktop

1. Open Claude Desktop -> Settings -> Developer -> Edit Config (`claude_desktop_config.json`)
2. Add the skill by placing the `SKILL.md` file in your Claude skills directory:

```bash
# macOS/Linux
mkdir -p ~/.claude/skills/seo-audit
curl -o ~/.claude/skills/seo-audit/SKILL.md \
  https://raw.githubusercontent.com/adityash8/claude-seo-audit-skill/main/.claude/skills/seo-audit/SKILL.md
```

```bash
# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\seo-audit"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/adityash8/claude-seo-audit-skill/main/.claude/skills/seo-audit/SKILL.md" `
  -OutFile "$env:USERPROFILE\.claude\skills\seo-audit\SKILL.md"
```

3. Restart Claude Desktop

### Claude Code (CLI)

```bash
mkdir -p ~/.claude/skills/seo-audit
curl -o ~/.claude/skills/seo-audit/SKILL.md \
  https://raw.githubusercontent.com/adityash8/claude-seo-audit-skill/main/.claude/skills/seo-audit/SKILL.md
```

The skill will be available in any Claude Code session automatically.

### Cursor

1. Open Cursor -> Settings -> Rules for AI (or `.cursorrules`)
2. Copy the contents of `SKILL.md` into your rules file, or reference it:

```bash
mkdir -p ~/.cursor/skills/seo-audit
curl -o ~/.cursor/skills/seo-audit/SKILL.md \
  https://raw.githubusercontent.com/adityash8/claude-seo-audit-skill/main/.claude/skills/seo-audit/SKILL.md
```

3. In Cursor's "Rules for AI" settings, add:
```
When the user asks for an SEO audit, follow the instructions in ~/.cursor/skills/seo-audit/SKILL.md
```

### Windsurf / Codeium

Place `SKILL.md` in your project's `.windsurf/` directory or reference it in your global rules:

```bash
mkdir -p ~/.windsurf/skills
curl -o ~/.windsurf/skills/seo-audit.md \
  https://raw.githubusercontent.com/adityash8/claude-seo-audit-skill/main/.claude/skills/seo-audit/SKILL.md
```

### Any Claude API Project (Custom Integration)

Prepend the skill content to your system prompt, or inject it dynamically:

```python
import anthropic

with open("seo-audit/SKILL.md") as f:
    skill_content = f.read()

client = anthropic.Anthropic()
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=8096,
    system=skill_content,
    messages=[{"role": "user", "content": "Run an SEO audit for example.com"}]
)
```

## Connecting Ahrefs MCP

Ahrefs is the recommended data source — it gives you real CrUX CWV data, per-page schema validation, internal link analysis, and live GSC data.

1. Go to [app.ahrefs.com](https://app.ahrefs.com) -> Account Settings -> API -> MCP
2. Enable the MCP server and copy your connection URL
3. Add to Claude Desktop config:

```json
{
  "mcpServers": {
    "ahrefs": {
      "command": "npx",
      "args": ["-y", "@ahrefs/mcp-server"],
      "env": {
        "AHREFS_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

4. In Ahrefs, connect Google Search Console to your project (for GSC-powered CTR and keyword data)
5. Note your **Site Audit project ID** from the URL: `app.ahrefs.com/site-audit/{PROJECT_ID}/...`

## Connecting Semrush MCP

**Official MCP (OAuth, no API key):**
Add to Claude Desktop config:
```json
{
  "mcpServers": {
    "semrush": {
      "command": "npx",
      "args": ["-y", "@semrush/mcp-server"]
    }
  }
}
```

**Community MCP (API key required):**
```json
{
  "mcpServers": {
    "semrush": {
      "command": "npx",
      "args": ["-y", "semrush-mcp"],
      "env": {
        "SEMRUSH_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

## Usage

Once installed, trigger the audit by asking Claude:

- `run an seo audit for example.com`
- `audit this site: example.com`
- `why is my traffic dropping?`
- `technical seo check for example.com`
- `seo deep dive on example.com`

Claude will ask which MCP tool you have connected, gather the required inputs (domain, project IDs, ARPU), then run all audit sections in parallel and deliver a structured report with:

1. Executive summary with health score and revenue opportunity
2. Quick wins, high-impact items, and ongoing recommendations
3. All 20 audit section results with pass/warn/critical status
4. Priority action matrix
5. Copy-paste fixes (robots.txt, JSON-LD blocks, meta tags, redirects)
6. GEO deliverables (TL;DR blocks, AI surface scores, CTR gap report)

## Audit Sections

| # | Section | Requires |
|---|---------|---------|
| 1 | Manual Actions (GSC penalties) | GSC link |
| 2 | Global Redirect | web_fetch |
| 3 | Organic Traffic | Ahrefs/Semrush |
| 4 | MoM Growth | Ahrefs/Semrush |
| 5 | Core Web Vitals | Ahrefs (CrUX) / PageSpeed API |
| 6 | Indexability | Ahrefs / web_search |
| 7 | Robots.txt | web_fetch |
| 8 | XML Sitemaps | web_fetch + Ahrefs |
| 9 | Canonicalization | Ahrefs |
| 10 | Content Decay | Ahrefs/Semrush |
| 11 | Broken Links | Ahrefs |
| 12 | HTTP Issues | Ahrefs |
| 13 | Schema Markup | Ahrefs |
| 14 | Internal Linking | Ahrefs |
| 15 | Backlink Profile | Ahrefs/Semrush |
| 16 | Content Gap | Ahrefs/Semrush |
| 17 | Organic CTR | Ahrefs GSC / Semrush |
| 18 | GEO TL;DR Blocks | web_fetch |
| 19 | AI Surface Scoring | web_search |
| 20 | Schema Page-Type Matrix | Ahrefs |

## Stack-Aware Fix Application

After the audit, Claude can automatically apply fixes if you have the relevant MCP connected:

| Stack | Automatable Fixes |
|-------|-----------------|
| Webflow Enterprise | robots.txt, 301 redirects, well-known files |
| Cloudflare | robots.txt, HSTS headers, HTTP->HTTPS redirects |
| Vercel | redirects, security headers in vercel.json |
| WordPress.com | page meta, redirects, schema injection |
| WordPress self-hosted | same as above via mcp-adapter plugin |
| Claude Code / custom | copy-paste diffs for any file |

## License

MIT — use freely, attribution appreciated.

## Contributing

PRs welcome for:
- Additional MCP tool support (Moz, Majestic, Screaming Frog)
- New audit sections (E-E-A-T signals, page experience, INP patterns)
- Stack-specific fix templates
- Non-English market adaptations
