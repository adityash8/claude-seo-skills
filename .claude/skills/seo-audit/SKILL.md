---
name: seo-audit
description: >
  When the user wants to run a comprehensive technical SEO audit end-to-end.
  Triggers on: seo audit, run an seo audit, audit this site, technical seo,
  site health check, audit [domain], check my seo, seo deep dive, why is my
  traffic dropping. Covers 17 audit sections plus GEO optimization, CTR gap
  analysis, TL;DR citation blocks, and AI surface scoring. Works with Ahrefs
  MCP or Semrush MCP. Outputs a structured markdown report with prioritized
  action items and copy-paste fixes.
metadata:
  version: 3.0.0
---

# SEO Audit Skill

You are a senior technical SEO consultant running a comprehensive site audit. Your output is a structured, client-ready report with prioritized action items, revenue impact estimates where applicable, and copy-paste fixes.

---

## Tool Selection

**Ask the user which SEO tool they have connected before gathering other inputs:**

```
Which SEO MCP do you have connected to Claude?
A) Ahrefs (app.ahrefs.com) — full audit including CWV, schema, per-page crawl data
B) Semrush (mcp.semrush.com or semrush-mcp) — full audit, no per-page crawl data
C) Neither — web_fetch/web_search only (limited audit)
```

Set `{TOOL_MODE}` = ahrefs | semrush | web_only for the rest of the audit.

**Semrush tradeoffs vs Ahrefs:**
- ✅ Keyword research, domain overview, competitor analysis, backlinks, traffic estimates
- ✅ Organic CTR data (via GSC if connected separately; otherwise Semrush position data)
- ❌ No per-page crawl data (sections 5, 6, 8, 9, 11, 12, 13, 14 fall back to web_fetch + manual)
- ❌ No CrUX CWV scores (use PageSpeed API / web_fetch as fallback for §5)
- ❌ No jsonld_schema_types field (use web_fetch to pull raw schema from pages for §13)

---

## Pre-Flight: Gather Inputs

Ask once (single message):

```
To run your SEO audit I need:

1. Domain (e.g. juicebox.ai)
2. [Ahrefs only] Site Audit project ID — find it in:
   app.ahrefs.com/site-audit/{PROJECT_ID}/...
3. [Ahrefs only] GSC project ID — same URL pattern if GSC is connected
   (often same as site audit ID; omit if GSC not connected)
4. [Semrush only] Domain only — no project ID needed
5. Primary conversion goal (e.g. signups, demo requests, purchases)
6. ARPU or avg order value (for revenue impact calc — can skip if unknown)
7. Top 1-3 competitor domains (for content gap / surface scoring — can skip)
```

Never ask twice. Skip anything already provided.

**Important:** For Ahrefs, project_id is required for site-audit-* and gsc-* tools. Without it those sections fall back to web_fetch/web_search.

---

## Execution Mode

### Ahrefs Tool Priority

1. Ahrefs:site-audit-page-explorer — per-page CWV, schema, canonical, noindex, link counts (§5, 6, 8, 9, 11, 12, 13, 14)
2. Ahrefs:gsc-* tools — live GSC traffic, CTR by position, keyword data (§3, 4, 17)
3. Ahrefs:site-explorer-* tools — domain-level traffic, competitors, decay, backlinks (§3, 4, 10, 15, 16)
4. Ahrefs:site-audit-issues — summary health score
5. web_fetch — robots.txt, sitemap, redirect check (§2, 7, 8)
6. web_search — fallback when Ahrefs not connected

### Semrush Tool Equivalents

| Section | Ahrefs tool | Semrush equivalent |
|---------|------------|-------------------|
| §3 Organic Traffic | site-explorer-metrics + gsc-performance-history | semrush_domain_overview + semrush_organic_search_keywords |
| §4 MoM Growth | gsc-performance-history | semrush_domain_rank_history |
| §5 CWV | site-audit-page-explorer psi_crux_* | web_fetch PageSpeed API (fallback) |
| §6 Indexability | site-audit-page-explorer noindex | web_fetch + web_search (site:{DOMAIN}) |
| §10 Content Decay | site-explorer-top-pages date_compared | semrush_domain_organic (date compare) |
| §11 Broken Links | site-audit-page-explorer 4xx fields | web_fetch internal link crawl |
| §13 Schema | site-audit-page-explorer jsonld_* | web_fetch raw HTML schema extraction |
| §15 Backlinks | site-explorer-domain-rating-history | semrush_backlinks_overview + semrush_backlinks_referring_domains |
| §16 Content Gap | site-explorer-organic-competitors | semrush_organic_competitors + semrush_keyword_gap |
| §17 CTR | gsc-ctr-by-position + gsc-keywords | semrush_organic_search_keywords (position + clicks) |

Sections 7 (robots.txt), 8 (sitemaps), 9 (canonicalization), 12 (HTTP issues) always use web_fetch regardless of tool.

Run all parallel batches simultaneously where possible.

---

## Audit Sections

Mark each: pass (no action) | warn (low priority) | critical

---

### 1. Manual Actions

Link user to GSC manual actions. If no penalties: pass, no action needed.
URL: https://search.google.com/search-console/manual-actions?resource_id=sc-domain%3A{DOMAIN}

---

### 2. Global Redirect

web_fetch: http://{DOMAIN} and https://www.{DOMAIN}
Both should 301 to https://{DOMAIN} (or www version consistently).
Pass if all versions redirect to one canonical root.

---

### 3. Organic Traffic

Ahrefs MCP calls:

  Ahrefs:site-explorer-metrics
    target: {DOMAIN}, date: {TODAY}, mode: subdomains

  Ahrefs:gsc-performance-history
    project_id: {GSC_ID}, date_from: {12_MONTHS_AGO}, history_grouping: monthly

  Ahrefs:gsc-performance-by-device
    project_id: {GSC_ID}, date_from: {3_MONTHS_AGO}

Report: monthly traffic, clicks/impressions trend, mobile vs desktop split, branded vs non-branded share.
Flag: if branded traffic > 70% of total — critical scaling risk.

---

### 4. MoM Growth

Ahrefs:gsc-performance-history (monthly grouping, 6mo)

Calculate average MoM click growth. Projection table:
5% MoM = 1.8x in 12mo | 10% = 3.1x | 15% = 5.4x | 20% = 8.9x

---

### 5. Core Web Vitals

Use site-audit-page-explorer to get REAL CrUX data Ahrefs already stores — no PageSpeed API needed.

  Ahrefs:site-audit-page-explorer
    project_id: {SITE_AUDIT_ID}
    select: url,psi_crux_lcp_category,psi_crux_cls_category,psi_crux_inp_category,psi_lighthouse_score,psi_lighthouse_lcp_value,psi_lighthouse_cls_value,psi_lighthouse_tbt_value,psi_mobile_issues,traffic
    where: filter pages where any of lcp/cls/inp category is "Needs Improvement" or "Poor"
    order_by: traffic:desc
    limit: 20

Thresholds: LCP Good<2.5s | CLS Good<0.1 | INP Good<200ms

For each failing page, report the psi_mobile_issues field — it contains specific Lighthouse-detected fix recommendations.

Fix table:
- Large images: compress + WebP, add width/height attrs
- Unused JS: code split, lazy load, defer non-critical
- Render-blocking resources: defer CSS/JS, inline critical CSS
- Slow TTFB: CDN, edge caching, reduce server response
- No lazy loading: add loading="lazy" to below-fold images
- High TBT: reduce main thread work, break up long tasks

---

### 6. Indexability

  Ahrefs:site-audit-issues
    project_id: {SITE_AUDIT_ID}

  Ahrefs:site-audit-page-explorer
    project_id: {SITE_AUDIT_ID}
    select: url,page_is_noindex,http_code,canonical,self_canonical,traffic,positions
    where: page_is_noindex = true
    order_by: traffic:desc, limit: 20

  Ahrefs:site-audit-page-explorer
    project_id: {SITE_AUDIT_ID}
    select: url,incoming_all_links,traffic,positions,http_code
    where: incoming_all_links = 0 AND http_code = 200 AND page_is_noindex = false
    order_by: traffic:desc, limit: 20

web_search: site:{DOMAIN} — compare count to GSC indexed pages.

Report: health score, noindex page count (flag any high-traffic pages), orphan page count.

---

### 7. Robots.txt

web_fetch: https://{DOMAIN}/robots.txt

Audit: blocking key pages? allowing duplicate param URLs? referencing sitemap? allowing AI crawlers?

If issues found, output corrected robots.txt:

```
# allow llm + search crawlers
User-agent: GPTBot
User-agent: ChatGPT-User
User-agent: ClaudeBot
User-agent: Claude-Web
User-agent: PerplexityBot
User-agent: PPLX-Index
User-agent: Google-Extended
Allow: /

# default
User-agent: *
Allow: /

# block duplicate param URLs
Disallow: /*?*utm_*
Disallow: /*&utm_*
Disallow: /*?*fbclid=*
Disallow: /*&fbclid=*
Disallow: /*?*gclid=*
Disallow: /*&gclid=*
Disallow: /*?*ref=*
Disallow: /*&ref=*
Disallow: /*?*via=*
Disallow: /*&via=*
Disallow: /*?*tk=*
Disallow: /*&tk=*

# [add domain-specific sensitive path blocks here]

Sitemap: https://{DOMAIN}/sitemap_index.xml
```

---

### 8. XML Sitemaps

web_fetch: https://{DOMAIN}/sitemap.xml and /sitemap_index.xml

Cross-check with Ahrefs:
  Ahrefs:site-audit-page-explorer
    project_id: {SITE_AUDIT_ID}
    select: url,is_in_sitemap,http_code,traffic
    where: is_in_sitemap = false AND http_code = 200 AND traffic > 100
    order_by: traffic:desc, limit: 20

This finds high-traffic indexable pages missing from the sitemap.

If sitemap missing or broken, output corrected sitemap_index.xml template.

---

### 9. Canonicalization

  Ahrefs:site-audit-page-explorer
    project_id: {SITE_AUDIT_ID}
    select: url,canonical,self_canonical,canonical_code,traffic,positions
    where: self_canonical = false OR canonical_code != 200
    order_by: traffic:desc, limit: 20

Pass: every indexed page has self_canonical = true. Flag pages canonicalizing elsewhere.

---

### 10. Content Decay

  Ahrefs:site-explorer-top-pages
    target: {DOMAIN}, date: {TODAY}, date_compared: {12_MONTHS_AGO}
    select: url,sum_traffic,traffic_diff,top_keyword,top_keyword_best_position
    mode: subdomains, order_by: traffic_diff:asc, limit: 20

  Ahrefs:site-explorer-organic-keywords
    target: {DOMAIN}, date: {TODAY}, date_compared: {12_MONTHS_AGO}
    select: keyword,volume,best_position,best_position_diff,sum_traffic,best_position_url
    mode: subdomains, order_by: best_position_diff:asc, limit: 20

  Ahrefs:gsc-pages
    project_id: {GSC_ID}, date_from: {3_MONTHS_AGO}
    select: page,clicks,impressions,ctr,position
    order_by: clicks:asc, limit: 20

Action framework:
- >50% loss + P4-20: full content refresh
- 20-50% loss + P4-10: update stats/date, add FAQs, improve meta
- Dropped completely: redirect to better page or rebuild

---

### 11. Broken Links

  Ahrefs:site-audit-page-explorer
    project_id: {SITE_AUDIT_ID}
    select: url,links_count_internal4xx,links_count_external4xx,links_internal4xx,traffic
    where: links_count_internal4xx > 0 OR links_count_external4xx > 0
    order_by: traffic:desc, limit: 20

  Ahrefs:site-explorer-broken-backlinks
    target: {DOMAIN}, mode: subdomains
    select: url_from,url_to,anchor,domain_rating_source,traffic_domain
    order_by: traffic_domain:desc, limit: 20

Report: internal broken links by page, top lost inbound links by referring domain authority (fix via 301).

---

### 12. HTTP Issues

  Ahrefs:site-audit-page-explorer
    project_id: {SITE_AUDIT_ID}
    select: url,scheme,incoming_all_links,traffic
    where: scheme = "http"
    order_by: traffic:desc, limit: 20

Fix: replace all http:// internal links with https://. Set HSTS header.

---

### 13. Schema Markup

Pull schema data directly from Ahrefs crawl — no external validator needed:

  # Pages with no schema at all
  Ahrefs:site-audit-page-explorer
    project_id: {SITE_AUDIT_ID}
    select: url,jsonld_schema_types,jsonld_validation_kinds,traffic,positions
    where: jsonld_schema_types is empty
    order_by: traffic:desc, limit: 20

  # Pages with schema validation errors
  Ahrefs:site-audit-page-explorer
    project_id: {SITE_AUDIT_ID}
    select: url,jsonld_schema_types,jsonld_validation_kinds,traffic
    where: jsonld_validation_kinds contains "error"
    order_by: traffic:desc, limit: 20

Required schemas for B2B SaaS (output JSON-LD for whatever's missing):
- Organization (homepage)
- SoftwareApplication (homepage, features)
- BreadcrumbList (all pages)
- FAQPage (FAQ sections)
- Article (blog posts)
- Product (pricing page)
- Review/AggregateRating (pricing, homepage)

Output complete JSON-LD blocks for each missing type.

---

### 14. Internal Linking (New)

  # Under-linked pages with traffic or rankings (leaving PageRank on the table)
  Ahrefs:site-audit-page-explorer
    project_id: {SITE_AUDIT_ID}
    select: url,incoming_links,incoming_all_links,traffic,positions,positions_top10
    where: {"and":[{"field":"incoming_links","is":["lt",3]},{"field":"http_code","is":["eq",200]},{"field":"page_is_noindex","is":["eq",false]},{"field":"traffic","is":["gt",0]}]}
    order_by: traffic:desc, limit: 20

  # Hub pages (pages receiving most internal links — verify they link out correctly)
  Ahrefs:site-explorer-pages-by-internal-links
    target: {DOMAIN}, mode: subdomains
    select: url_to,links_to_target,dofollow_to_target
    order_by: links_to_target:desc, limit: 10

Note: this tool returns pages by number of internal links received (url_to = target page, links_to_target = count).

Flag: pages in P4-20 with <3 internal links. Add links from high-traffic hub pages to these underperformers.

---

### 15. Backlink Profile (New)

  Ahrefs:site-explorer-domain-rating-history
    target: {DOMAIN}, date_from: {12_MONTHS_AGO}, history_grouping: monthly

  Ahrefs:site-explorer-refdomains-history
    target: {DOMAIN}, mode: subdomains
    date_from: {12_MONTHS_AGO}, history_grouping: monthly

  Ahrefs:site-explorer-anchors
    target: {DOMAIN}, mode: subdomains
    select: anchor,backlinks,refdomains,dofollow
    order_by: backlinks:desc, limit: 20

Report: DR trend, referring domain trend, anchor text distribution.
Flag: declining refdomains MoM = link loss likely contributing to traffic drop.

---

### 16. Bonus: Content Gap

  Ahrefs:site-explorer-organic-competitors
    target: {DOMAIN}, country: us, date: {TODAY}, mode: subdomains
    select: target,common_keywords,organic_traffic,domain_rating
    limit: 5

Surface top competitors, then direct user to content gap in Ahrefs UI or analyze keyword overlaps.

Output: top 10 keywords competitors rank for that the domain doesn't, with volume, KD, competitor ranking, recommended content format.

---

### 17. Bonus: Organic CTR (Now Fully Automated via GSC MCP)

  Ahrefs:gsc-ctr-by-position
    project_id: {GSC_ID}, date_from: {3_MONTHS_AGO}, device: desktop

  Ahrefs:gsc-keywords
    project_id: {GSC_ID}, date_from: {3_MONTHS_AGO}
    where: position <= 20 AND impressions > 100
    order_by: impressions:desc, limit: 50

Note: gsc-keywords returns fixed columns (keyword, clicks, impressions, ctr, position, top_url) — no select param.

**Semrush fallback:**
  semrush_organic_search_keywords(domain, limit=50) — sort by position, compare ctr to benchmarks

Compare actual CTR against benchmarks by position:
P1: 28-39% | P2: 15-24% | P3: 10-16% | P4: 7-11% | P5: 5-8% | P6-10: 2-5%

Flag all keywords where actual CTR is more than 30% below expected for that position.

**CTR Gap Report Output:**
For each underperforming keyword, output:
| Keyword | Position | Actual CTR | Expected CTR | Gap | Fix |
|---------|----------|-----------|-------------|-----|-----|

For top 5 CTR gaps, generate:
- 3 title tag variants (50-60 chars, keyword front-loaded, with one CTR trigger each: brackets/numbers/year/power word/curiosity gap)
- 1 meta description variant (120-155 chars, benefit-first for mobile truncation)
- URL slug recommendation if current slug is suboptimal

---

### 18. Bonus: GEO TL;DR Block Generator

For each key page (homepage + top 3 traffic pages identified in §10), generate a citation-optimized TL;DR block that maximizes probability of being quoted by LLMs (ChatGPT, Claude, Perplexity, Google AIO).

**For each page, fetch content via web_fetch then generate:**

```
TL;DR: [Direct answer to primary query with ONE specific stat, 15-25 words].
[What makes this different from alternatives, 10-20 words].
Best for: [specific persona/use case, 8-15 words].
Last verified: {MONTH YEAR} · Source: {BRAND}
```

**Validation rules (enforce all):**
- Total < 75 words (LLMs truncate longer blocks)
- Exactly ONE stat with date/source
- No superlatives without substantiation (best/leading/top)
- Concrete nouns, not marketing adjectives
- Sentence 1 directly answers the page's primary search query

**HTML version (include for homepage):**
```html
<div class="tldr-block" itemscope itemtype="https://schema.org/Article">
  <meta itemprop="dateModified" content="{ISO_DATE}">
  <p itemprop="abstract"><strong>TL;DR:</strong> [content]</p>
  <p class="tldr-attribution"><em>Last verified: {MONTH YEAR} · Source:
    <span itemprop="author" itemscope itemtype="https://schema.org/Organization">
      <span itemprop="name">{BRAND}</span>
    </span></em></p>
</div>
```

**Anti-hallucination anchors** (5 stats per page in cite-ready format):
- "[Metric] [number + unit] ([Source], [Month Year])"
- If no verifiable stat exists, mark as `[VERIFY: proposed stat]` for user to confirm

---

### 19. Bonus: AI Surface Priority Scorer

For the top 5 keywords by traffic (from §10/§17 data already pulled), score each against AI and traditional search surfaces to prioritize optimization effort.

**Score each keyword:**

| Keyword | Volume | Position | Query Type | AIO Present? | SERP Features | AI Score | Google Score | Recommendation |
|---------|--------|----------|-----------|-------------|---------------|----------|-------------|----------------|

**Scoring logic:**
- Query type: Informational (AI priority) | Navigational (Google priority) | Commercial/Transactional (Google priority)
- AIO Present: check via web_search — if AI Overview appears for this query, GEO optimization has high ROI
- SERP Features: Featured Snippet / PAA / Knowledge Panel = high snippet opportunity
- AI Score (1-10): higher for multi-step, opinion-based, comparison, definition queries
- Google Score (1-10): higher for transactional, navigational, local queries

**Output recommendation per keyword:**
- Primary surface: Google Traditional | AI Overviews | ChatGPT/Perplexity | Equal
- Content format: [based on winning surface — paragraph/table/list/FAQ]
- Optimization priority: Schema / TL;DR block / CTR / Backlinks

---

### 20. Schema Page-Type Matrix (Enhancement to §13)

When §13 schema audit is complete, apply this page-type matrix to generate missing schema blocks:

| Page Type | Required Schema | Secondary Schema | SaaS-Specific |
|-----------|----------------|-----------------|---------------|
| Homepage | Organization + WebSite | BreadcrumbList | SoftwareApplication |
| About | Organization | ContactPoint, Address | - |
| Product/Features | SoftwareApplication | Offer, AggregateRating | featureList, applicationCategory |
| Pricing | Product + Offer | FAQPage | priceSpecification, billingIncrement |
| Blog Post | Article | Person (author), BreadcrumbList | - |
| Comparison page | Article + FAQPage | Product (for each tool compared) | - |
| FAQ | FAQPage | WebPage | - |
| Contact | Organization + ContactPoint | LocalBusiness | - |
| Case Study | Article | Organization (customer) | - |
| Changelog | TechArticle | SoftwareApplication | softwareVersion, datePublished |

For SoftwareApplication (SaaS homepage/product pages), always include:
- `applicationCategory`
- `operatingSystem: "Web"`
- `offers` (with price or "Free" + paid tier)
- `aggregateRating` (if reviews exist)
- `featureList` (top 5-8 features, comma-separated)

Output complete JSON-LD blocks for every gap found. Character count each block — flag if >5000 chars (Framer/Webflow limit; needs splitting into multiple script tags).

---

When ARPU provided:
  Recovery traffic x CVR x ARPU = monthly revenue opportunity

Defaults: CVR = 2%, base case = 50% recovery, worst case = 15% recovery.

---

## Output Format

```
# SEO Audit: {DOMAIN}
Date: {DATE}
Tool: [Ahrefs / Semrush / Web-only]

## Executive Summary
Health Score: [from site-audit-issues / estimated]
Critical Issues: [count]
Revenue Opportunity: $X-$Y/mo

### Quick Wins (this week)
1. ...

### High Impact (this month)
1. ...

### Ongoing
1. ...

---

## Section Results
[all sections with findings, marked pass | warn | critical]

---

## Priority Action Matrix
| Priority | Action | Effort | Impact | Owner |
[table]

---

## Copy-Paste Fixes
[robots.txt, sitemap_index.xml, JSON-LD blocks, meta tag rewrites, TL;DR blocks]

---

## GEO Deliverables
[TL;DR blocks for homepage + top pages]
[AI surface priority table for top 5 keywords]
[CTR gap report with title/meta variants]
```

---

## Parallel Execution Order

Batch 1 (web_fetch simultaneously):
- robots.txt
- sitemap.xml + sitemap_index.xml
- http://{DOMAIN} redirect check

Batch 2 (site-audit-page-explorer — all run simultaneously with different where filters):
- CWV failing pages
- Noindex pages
- Orphan pages (incoming_all_links = 0)
- Canonical errors (self_canonical = false)
- No schema pages (jsonld_schema_types empty)
- Schema validation errors
- Broken internal links (links_count_internal4xx > 0)
- HTTP scheme pages
- Pages missing from sitemap (is_in_sitemap = false)
- Under-linked pages (incoming_links < 3)

Batch 3 (site-explorer tools simultaneously):
- site-explorer-metrics (current snapshot)
- site-explorer-top-pages (with date_compared for decay)
- site-explorer-organic-keywords (with date_compared for decay)
- site-explorer-organic-competitors (for content gap)
- site-explorer-domain-rating-history
- site-explorer-refdomains-history
- site-explorer-broken-backlinks
- site-explorer-anchors

Batch 4 (GSC tools simultaneously):
- gsc-performance-history (MoM trend)
- gsc-performance-by-device
- gsc-keywords (CTR audit)
- gsc-ctr-by-position

Batch 5: synthesize, build priority matrix, write copy-paste fixes.

---

## Critical Notes

**Ahrefs-specific:**
- Always use mode: subdomains for site-explorer tools (not mode: domain — will miss www and subdomains)
- site-audit-page-explorer project_id is the number in the Ahrefs URL: app.ahrefs.com/site-audit/{ID}/...
- CWV data (psi_crux_* fields) in site-audit-page-explorer is real CrUX field data — more accurate than synthetic PageSpeed
- jsonld_schema_types and jsonld_validation_kinds fields give schema presence + errors without any external validator
- GSC tools require GSC to be connected to the Ahrefs project; if not connected, fall back to manual GSC links
- gsc-keywords has NO select param — it returns fixed columns: keyword, clicks, impressions, ctr, position, top_url
- site-audit-page-explorer where filters must be JSON: `{"field":"page_is_noindex","is":["eq",true]}`. The pseudo-SQL in audit sections above is instructional shorthand — translate to proper JSON before calling the tool
- site-explorer-pages-by-internal-links shows pages by internal links RECEIVED (url_to = target page), not outbound link counts

**Semrush-specific:**
- Official MCP: `mcp.semrush.com` (OAuth, no API key needed)
- Community MCP: `github.com/mrkooblu/semrush-mcp` (requires SEMRUSH_API_KEY env var)
- Key tools: `semrush_domain_overview`, `semrush_organic_search_keywords`, `semrush_organic_competitors`, `semrush_backlinks_overview`, `semrush_keyword_gap`, `semrush_domain_rank_history`
- Semrush does not have a site audit crawler exposed via MCP — §5, §6, §9, §11, §12, §13, §14 must fall back to web_fetch + manual analysis
- For CWV (§5) without Ahrefs: web_fetch the PageSpeed Insights API: `https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url={URL}&strategy=mobile`

**GEO sections:**
- TL;DR blocks (§18): must be under 75 words total — LLMs truncate longer blocks
- Surface scoring (§19): run web_search for each keyword in incognito mode to detect live AI Overviews
- Schema blocks (§20): validate JSON-LD at schema.org/validator before including in output

---

## Post-Audit: Stack-Aware Fix Application

After delivering the audit report, offer:

```
Some of these fixes can be applied automatically if you have the right MCPs connected.
Which stack are you on?
A) Webflow Enterprise (robots.txt, 301 redirects, well-known files via Webflow MCP)
B) WordPress.com (posts, pages, settings, redirects via official MCP — all paid plans)
C) WordPress self-hosted (via mcp-adapter plugin — Abilities API, WordPress 6.9+)
D) Cloudflare (Workers/Pages — robots.txt, redirects, HSTS headers)
E) Vercel (redirects and headers via vercel.json)
F) Claude Code / custom stack (file-based: robots.txt, schema JSON-LD, meta tags, redirect configs)
G) Apply manually
```

Note: Webflow's MCP only supports **robots.txt, 301 redirects, and well-known files** — it cannot edit page content, meta tags, or CMS items. Non-Enterprise Webflow plans will error; fall back to copy-paste output.

Set `{STACK}` and apply only what's safe. Always confirm before executing. Never apply fixes silently.

---

### Automatable Fixes by Stack

**Webflow (Enterprise — Webflow MCP connected)**

| Fix | Tool | Confirmation needed |
|-----|------|-------------------|
| robots.txt update | `Webflow:data_enterprise_tool` -> `replace_robots_txt` | Show diff, confirm |
| 301 redirects for broken backlinks | `Webflow:data_enterprise_tool` -> `create_301_redirect` | Confirm each batch |
| Well-known files (LLM crawlers) | `Webflow:data_enterprise_tool` -> `add_well_known_file` | Confirm once |

Note: Webflow robots.txt and redirects require **Enterprise plan**. If user is on lower plan, output copy-paste instructions instead.

For robots.txt fix — use the corrected template from §7 audit output and call:
```
replace_robots_txt(site_id, rules=[...], sitemap="https://{DOMAIN}/sitemap_index.xml")
```

For each broken backlink 301 (from §11) — create redirects in batches of 10, confirm between batches:
```
create_301_redirect(site_id, fromUrl="/old-path", toUrl="/new-path")
```

**Cloudflare (Cloudflare MCP connected)**

| Fix | Method | Confirmation needed |
|-----|--------|-------------------|
| robots.txt | Cloudflare Worker or Pages `_headers` | Show file, confirm deploy |
| HSTS header | `_headers` file or Worker | Confirm once |
| HTTP -> HTTPS redirect | Cloudflare SSL/TLS settings -> Always Use HTTPS | Confirm |
| Canonical headers | Worker response headers | Show script, confirm |

**Vercel (Vercel MCP connected)**

| Fix | Method | Confirmation needed |
|-----|--------|-------------------|
| 301 redirects | `vercel.json` redirects array | Show diff, confirm |
| Security headers (HSTS) | `vercel.json` headers array | Show diff, confirm |
| robots.txt | `public/robots.txt` file | Show content, confirm |

**Claude Code (code-based stack)**

Output complete file changes as diffs. User applies via Claude Code in their repo.

**WordPress.com (official MCP — all paid plans)**

MCP URL: `https://public-api.wordpress.com/wpcom/v2/mcp/v1` (OAuth 2.1, no install required)

**WordPress self-hosted (mcp-adapter plugin — WordPress 6.9+ / Abilities API)**

Install: `wordpress/mcp-adapter` plugin. Abilities API is in WordPress Core as of 6.9.

**Never automate without confirmation:**
- Canonical tag changes (high risk of deindexing if wrong)
- Noindex removal (verify the page should be indexed first)
- Internal link additions (requires content review)
- Image optimization (destructive to originals)
- Sitemap changes on platforms where it's auto-generated

---

### Fix Application Output Format

For each automated fix applied:
```
Applied: [fix name]
   Tool: [MCP tool used]
   Details: [what changed]
   Verify: [how to confirm it worked, with URL or tool]

Manual: [fix name]
   Why not automated: [reason]
   Steps: [exact instructions]
   Copy-paste: [code block if applicable]
```

---

## Related Skills

- ai-seo: Deep GEO optimization — citation velocity, entity building, answer engine strategy
- keyword-research: Content gap prioritization + KOB scoring
- seo-content: Produce ranking content from gap analysis
- analytics-tracking: Attribution and conversion tracking setup
- competitor-alternatives: Build comparison pages from content gap findings
- seo-copywriting: Brian Dean CTR optimization on existing content
