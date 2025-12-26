TOOLS: research-bus
VERSION: 1.0
UPDATED: 2025-12-25
COUNT: 12
STATUS: Production (Implemented)

---

## CORE RESEARCH (4 tools)

---

TOOL: research_query
STATUS: ✅ Implemented
INPUT: { query: string (required), backend?: "auto"|"direct_perplexity"|"comet_flow" (default: "auto"), model?: "sonar-small"|"sonar"|"sonar-pro" (default: "sonar-small"), importance?: "low"|"medium"|"high" (default: "medium"), citationMode?: "vault"|"full"|"none" (default: "vault"), maxTokens?: number (default: 1000), searchDomainFilter?: string[], searchRecencyFilter?: "day"|"week"|"month"|"year" }
OUTPUT: { success: boolean, research_id: string, query: string, response: string, citations: Citation[], metadata: { backend_used: string, cached: boolean, cost: number, quality_score: number, latency_ms: number, strategy_reason?: string }, references?: { vault_id: string, citation_count: number }, budgetStatus: BudgetCheck, error?: string }
USE: Execute research query via Perplexity API with intelligent caching and citation vaulting
EXAMPLE: research_query({ query: "latest developments in AI agents 2025", importance: "high" })
NOTES: Returns vault_id for later citation retrieval. Budget gating rejects low-importance queries when budget is tight.

---

TOOL: get_budget_status
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { daily_budget: number, used_today: number, remaining: number, percentage_used: number, reset_time: timestamp, queries_today: number, avg_cost_per_query: number, by_importance: { low: number, medium: number, high: number } }
USE: Get current budget status and usage breakdown by importance level
EXAMPLE: get_budget_status({})
NOTES: Budget resets at midnight local time.

---

TOOL: get_cache_stats
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { total_cached: number, hit_rate: number, hits: number, misses: number, size_bytes: number, oldest_entry: timestamp, newest_entry: timestamp, savings_estimate: number }
USE: Get cache performance statistics and estimated cost savings
EXAMPLE: get_cache_stats({})
NOTES: savings_estimate is approximate cost saved by cache hits.

---

TOOL: health_check
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { success: boolean, backends: { perplexity: HealthStatus, comet: HealthStatus, looker: HealthStatus }, synthesizer: SynthesizerConfig, timestamp: number }
USE: Check availability and latency of all backends including Looker
EXAMPLE: health_check({})
NOTES: Tests actual connectivity to all configured backends.

---

## CITATION VAULT (5 tools)

---

TOOL: get_citations
STATUS: ✅ Implemented
INPUT: { vault_id: string (required) }
OUTPUT: { success: boolean, vault_id: string, citations: Citation[], error?: string }
USE: Retrieve full citations from a vault by ID
EXAMPLE: get_citations({ vault_id: "cv_abc123" })
NOTES: Returns error if vault expired or not found. Vaults expire after 4 hours by default.

---

TOOL: get_citation_summary
STATUS: ✅ Implemented
INPUT: { vault_id: string (required) }
OUTPUT: { success: boolean, vault_id: string, count: number, domains: string[], created_at: timestamp, expires_at: timestamp, error?: string }
USE: Get citation count and domains without full content (lower token cost)
EXAMPLE: get_citation_summary({ vault_id: "cv_abc123" })
NOTES: Useful for checking if citations are still relevant without fetching full content.

---

TOOL: get_vault_stats
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { success: boolean, total_vaults: number, active_vaults: number, expired_vaults: number, total_citations: number, tokens_saved: number, avg_ttl_remaining: number }
USE: Get overall vault statistics including token savings
EXAMPLE: get_vault_stats({})
NOTES: tokens_saved estimates context tokens saved by using vault references.

---

TOOL: list_vaults
STATUS: ✅ Implemented
INPUT: { include_expired?: boolean (default: false), limit?: number (default: 20) }
OUTPUT: { success: boolean, vaults: VaultInfo[], count: number }
USE: List citation vaults with optional expiry filter
EXAMPLE: list_vaults({ include_expired: false, limit: 10 })
NOTES: Returns vault IDs, creation time, expiry time, and citation counts.

---

TOOL: clear_expired_vaults
STATUS: ✅ Implemented
INPUT: {}
OUTPUT: { success: boolean, cleared_count: number }
USE: Remove expired citation vaults to free storage
EXAMPLE: clear_expired_vaults({})
NOTES: Runs automatically every 5 minutes, but can be triggered manually.

---

## ENHANCEMENT (3 tools)

---

TOOL: optimize_query
STATUS: ✅ Implemented
INPUT: { query: string (required) }
OUTPUT: { success: boolean, original: string, optimized: string, synonyms_added: string[], suggestions: string[], complexity_score: number }
USE: Analyze and improve a query before execution
EXAMPLE: optimize_query({ query: "AI trends" })
NOTES: Adds relevant synonyms, suggests refinements, and assesses query complexity.

---

TOOL: validate_source
STATUS: ✅ Implemented
INPUT: { url?: string, vault_id?: string }
OUTPUT: { success: boolean, url?: string, vault_id?: string, credibility_score: number, source_type: string, bias_indicator?: string, aggregate?: AggregateCredibility, error?: string }
USE: Check source credibility via Looker integration
EXAMPLE: validate_source({ url: "https://example.com/article" })
NOTES: Can validate single URL or all sources in a vault. Requires Looker server running.

---

TOOL: synthesize_results
STATUS: ✅ Implemented
INPUT: { query: string (required), research_ids: string[] (required), style?: "summary"|"comparative"|"comprehensive" (default: "summary"), maxLength?: number (default: 500) }
OUTPUT: { success: boolean, synthesis: string, sources_used: number, style: string, quality_metrics: { coverage: number, coherence: number, evidence_strength: number }, error?: string }
USE: Combine multiple research results into a cohesive summary using Claude API
EXAMPLE: synthesize_results({ query: "Compare AI agent frameworks", research_ids: ["rq_abc", "rq_def"], style: "comparative" })
NOTES: Requires ANTHROPIC_API_KEY. Uses Claude to synthesize findings across multiple research results.

---

## CONFIGURATION

CONFIG_FILE: config/research-bus-config.json

```json
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
```

---

## HTTP ADMIN ENDPOINTS

PORT: 8019

GET /health - Server health check
GET /budget - Budget status
GET /cache - Cache statistics
GET /vaults - List vaults
POST /vaults/clear - Clear expired vaults
GET /research/:id - Get research event by ID

---

## TYPE DEFINITIONS

```typescript
interface Citation {
  url: string;
  title: string;
  snippet?: string;
  domain: string;
}

interface VaultInfo {
  vault_id: string;
  created_at: number;
  expires_at: number;
  citation_count: number;
}

interface HealthStatus {
  available: boolean;
  latency_ms?: number;
  last_error?: string;
}

interface BudgetCheck {
  allowed: boolean;
  reason?: string;
  remaining: number;
  percentage_used: number;
}
```
