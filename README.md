# Claude SEO Skills

A collection of Claude skills for technical SEO, content strategy, and AI search optimization.

---

## Available Skills

### `seo-audit` — Comprehensive Technical SEO Audit

A senior technical SEO consultant in a prompt. Runs a structured 20-section audit against your domain and delivers a client-ready report with prioritized fixes, copy-paste code, and revenue impact estimates.

**What it covers:**
- **20 audit sections**: Manual actions, global redirect, organic traffic, MoM growth, Core Web Vitals, indexability, robots.txt, XML sitemaps, canonicalization, content decay, broken links, HTTP issues, schema markup, internal linking, backlink profile, content gap, CTR analysis, GEO TL;DR blocks, AI surface scoring, schema page-type matrix
- **GEO optimization**: Generates citation-optimized TL;DR blocks (<75 words) for LLM/AI visibility (ChatGPT, Perplexity, Google AIO)
- **AI surface scoring**: Scores each keyword against Google Traditional vs AI surfaces and recommends content format per surface
- **CTR gap analysis**: Compares actual CTR to position benchmarks, generates 3 title variants + meta description for each gap
- **Stack-aware fix application**: Auto-applies fixes for Webflow Enterprise, Cloudflare, Vercel, WordPress — or outputs copy-paste diffs
- **Revenue impact estimates**: Calculates monthly opportunity from traffic recovery (CVR x ARPU x recovered traffic)

**Works with:** Ahrefs MCP (full audit) | Semrush MCP (domain-level) | web_fetch only (limited)

**Install:**
```bash
mkdir -p ~/.claude/skills/seo-audit && curl -o ~/.claude/skills/seo-audit/SKILL.md \
  https://raw.githubusercontent.com/adityash8/claude-seo-skills/main/.claude/skills/seo-audit/SKILL.md
```

**Trigger phrases:** `seo audit`, `audit this site`, `why is my traffic dropping`, `technical seo`, `check my seo`

---

## More Skills Coming Soon

- `ai-seo` — Deep GEO optimization: citation velocity, entity building, answer engine strategy
- `keyword-research` — Content gap prioritization + KOB scoring
- `seo-content` — Produce ranking content from keyword gap analysis
- `analytics-tracking` — Attribution and conversion tracking setup
- `competitor-alternatives` — Build comparison pages from content gap findings

---

## Setup

### Claude Desktop

Place the skill file in `~/.claude/skills/{skill-name}/SKILL.md` and restart Claude Desktop.

### Claude Code (CLI)

```bash
mkdir -p ~/.claude/skills/seo-audit && curl -o ~/.claude/skills/seo-audit/SKILL.md \
  https://raw.githubusercontent.com/adityash8/claude-seo-skills/main/.claude/skills/seo-audit/SKILL.md
```

### Cursor

Copy the skill content into `.cursorrules` or reference it in Cursor's "Rules for AI" settings.

### Windsurf / Codeium

Place in `.windsurf/skills/` or reference in global rules.

### Claude API

Prepend the skill content to your system prompt:

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

1. Go to [app.ahrefs.com](https://app.ahrefs.com) → Account Settings → API → MCP
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

4. Connect Google Search Console to your Ahrefs project for GSC-powered CTR and keyword data
5. Note your **Site Audit project ID** from the URL: `app.ahrefs.com/site-audit/{PROJECT_ID}/...`

## Connecting Semrush MCP

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

## License

MIT — use freely, attribution appreciated.
