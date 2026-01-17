SERVER: research-bus
VERSION: 1.1
UPDATED: 2026-01-16
STATUS: Production
PORT: 8019 (HTTP only)
MCP: stdio transport (stdin/stdout JSON-RPC)
UDP: None (no InterLock mesh)
DEPS: Perplexity API
PURPOSE: Unified external intelligence gateway - 12-tool research interface with Perplexity API, Citation Vault, and result enhancement
CONFIG: /repo/research-bus/src/config.ts

---

ARCHITECTURE

FUNCTION: External research queries with intelligent backend selection
BACKENDS: Perplexity (direct), Comet Flow (workflow)
BUDGET: Daily budget management with importance-based gating
CACHE: Query result caching with normalization for cost efficiency
CITATIONS: Vault system for efficient citation reference without re-fetching
ENHANCEMENT: Query optimization, source validation, result synthesis

---

TOOL CATEGORIES (12 total)

CORE_RESEARCH (4):
- research_query: Execute research via Perplexity with caching and citation vaulting
- get_budget_status: Current budget, usage, and spending breakdown
- get_cache_stats: Cache performance metrics and cost savings
- health_check: Backend availability and latency status

CITATION_VAULT (5):
- get_citations: Retrieve full citations from vault by ID
- get_citation_summary: Get count/domains without full content
- get_vault_stats: Overall vault statistics (count, tokens saved)
- list_vaults: List vaults with optional expiry filter
- clear_expired_vaults: Remove expired vaults to free storage

ENHANCEMENT (3):
- optimize_query: Improve query before execution with synonym expansion
- validate_source: Check source credibility via Looker integration
- synthesize_results: Combine multiple results with Claude API

---

KEY CONCEPTS

BACKEND: External research provider (Perplexity direct, Comet Flow)
BUDGET: Daily spending limit with importance-based gating (low/medium/high)
CACHE: Normalized query results stored to avoid duplicate API calls
VAULT: Temporary citation storage with configurable TTL (4 hours default)
IMPORTANCE: Query priority affecting backend selection and budget gates

---

WORKFLOW

1. RECEIVE: Query arrives with importance level and options
2. BUDGET_CHECK: Verify query allowed under current budget
3. CHECK_CACHE: Return cached result if available and fresh
4. SELECT_BACKEND: Choose backend based on query type and budget
5. EXECUTE: Send query to selected backend (Perplexity or Comet)
6. PROCESS_CITATIONS: Store citations in vault, return vault_id
7. CACHE: Store normalized result for future queries
8. RETURN: Response with vault_id reference and metadata

---

BACKEND SELECTION

| Importance | Budget Available | Backend |
|------------|------------------|---------|
| low        | any              | cache or sonar |
| medium     | >50%             | sonar |
| medium     | <50%             | cache only or reject |
| high       | any              | sonar-pro or comet_flow |

---

INTEGRATIONS

PERPLEXITY: Direct API (sonar, sonar-pro, sonar-reasoning-pro, sonar-deep-research)
COMET_FLOW: Browser automation via CDP for web scraping research
LOOKER: Source credibility validation (port 8006)
CHRONOS_SYNAPSE: Query event tracking
CLAUDE_API: Result synthesis for multi-result summaries

---

HTTP API (port 8019)

| Endpoint | Method | Description |
|----------|--------|-------------|
| /health | GET | Backend status, versions, latency |
| /api/budget | GET | Budget status, spending, limits |
| /api/cache | GET | Cache stats - hits, misses, size |
| /api/cache/clear | POST | Clear all cached results |
| /api/vaults | GET | List citation vaults |
| /api/vaults/:id | GET | Get vault with citations |
| /api/vaults/clear-expired | POST | Remove expired vaults |
| /api/citations | GET | Get citations by vault_id |
| /api/optimize | POST | Optimize query before execution |
| /api/validate | POST | Validate source credibility via Looker |
| /api/research | POST | Execute research query |
| /api/synthesize | POST | Synthesize texts with Claude |
| /api/stats | GET | Combined stats (vault, cache, budget) |
| /api/history | GET | Recent research history |
| /api/tools | GET | List all MCP tools (bop-gateway) |
| /api/tools/:toolName | POST | Execute MCP tool via HTTP (bop-gateway) |

RESEARCH_QUERY_PARAMS:
- query: Search query (required)
- backend: auto, direct_perplexity, comet_flow
- model: sonar, sonar-pro, sonar-reasoning-pro, sonar-deep-research
- importance: low, medium, high
- max_citations: Number of citations to return

SYNTHESIZE_PARAMS:
- query: Context question (required)
- texts: Array of texts to synthesize (required, min 2)
- style: summary, comparative, comprehensive
- max_length: Maximum synthesis length

---

CONFIGURATION

CONFIG_FILE: config/research-bus-config.json

{
  "daily_budget": 10.00,
  "backends": {
    "perplexity": {
      "api_key": "env:PERPLEXITY_API_KEY",
      "default_model": "sonar-small",
      "timeout_ms": 30000
    },
    "comet_flow": {
      "endpoint": "http://localhost:8020",
      "enabled": false
    }
  },
  "cache": {
    "enabled": true,
    "ttl_hours": 24,
    "max_size_mb": 100
  },
  "vault": {
    "ttl_hours": 4,
    "cleanup_interval_min": 5
  },
  "admin": {
    "httpPort": 8019,
    "enabled": true
  }
}

---

TYPE DEFINITIONS

Citation: { url, title, snippet?, domain }
VaultInfo: { vault_id, created_at, expires_at, citation_count }
HealthStatus: { available, latency_ms?, last_error? }
BudgetCheck: { allowed, reason?, remaining, percentage_used }

---

KEY FILES

SOURCE: /repo/research-bus/
INDEX: src/index.ts
DATABASE: src/database.ts
BUDGET: src/budget.ts
CACHE: src/cache.ts
CITATIONS: src/citation-processor.ts
STRATEGY: src/strategy/selector.ts
ENHANCEMENT: src/enhancement/*.ts
ADMIN: src/admin/http-server.ts
CONFIG: config/research-bus-config.json

---

COMPARISON TO NIWS-SERVER

| Aspect | research-bus | niws-server |
|--------|--------------|-------------|
| Purpose | Research queries | News production pipeline |
| Tools | 12 | 100+ |
| Scope | Query → Response | RSS → Video delivery |
| Status | Production | Production |
| Port | 8019 (HTTP only) | 3015/8015/9015 |
