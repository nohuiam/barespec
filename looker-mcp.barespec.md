SERVER: looker-mcp
VERSION: 2.0.0
UPDATED: 2026-01-16
STATUS: Production
PORT: 8006 (HTTP only)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: Enhanced search with credibility validation and source scoring
CONFIG: /repo/looker-mcp/

---

ARCHITECTURE

SEARCH: Multi-source search aggregation with credibility scoring
CREDIBILITY: Domain-based scoring, content type detection
CACHING: In-memory search result caching
LAYERS: MCP stdio (5 tools), HTTP REST

---

TOOLS (5)

TOOL: enhanced_search
INPUT: { query: string (required), sources?: string[] (default: ["all"]), max_results?: number (default: 10) }
OUTPUT: { results: [{ url, title, snippet, score, credibility }], total, search_time_ms }
USE: Search with credibility-enhanced results
EXAMPLE: enhanced_search({ query: "AI safety research", max_results: 20 })

TOOL: validate_source
INPUT: { url: string (required) }
OUTPUT: { url, domain, credibility: { score, tier, signals, warnings }, metadata }
USE: Validate credibility of a URL source
EXAMPLE: validate_source({ url: "https://arxiv.org/abs/2405.00732" })

TOOL: get_domain_credibility
INPUT: { domain: string (required) }
OUTPUT: { domain, credibility_score, content_type, flags }
USE: Get credibility score for a domain
EXAMPLE: get_domain_credibility({ domain: "arxiv.org" })

TOOL: analyze_content_type
INPUT: { url: string (required) }
OUTPUT: { url, content_type, confidence }
USE: Detect content type (academic, news, blog, social_media, etc.)
EXAMPLE: analyze_content_type({ url: "https://example.com/article" })

TOOL: get_search_stats
INPUT: {}
OUTPUT: { cache: { entries, maxSize }, searches_performed }
USE: Get search cache statistics
EXAMPLE: get_search_stats({})

---

CREDIBILITY TIERS

- high (>= 0.7): Trusted academic, government, established news
- medium (0.4 - 0.7): General reputable sources
- low (< 0.4): User-generated, unverified sources
- unknown (0): Unscored domains

---

HTTP REST API (Port 8006)

GET  /health                    → { status, server, version, port, cache, timestamp }
GET  /api/tools                 → { tools: [], count: number } (Gateway integration)
POST /api/tools/:toolName       → { success: boolean, result: object } (Gateway integration)
GET  /stats                     → { success, cache, timestamp }
POST /api/credibility           → Validate URL credibility
POST /api/search                → Enhanced search

---

KEY FILES

SOURCE: /repo/looker-mcp/
INDEX: src/index.ts
SCORING: src/scoring/credibilityScorer.ts
ENGINES: src/engines/searchEngine.ts
HTTP: src/server/httpServer.ts
UTILS: src/utils/logger.ts

DEPENDENCIES: @modelcontextprotocol/sdk, express, zod, uuid
