SERVER: looker
VERSION: 2.0
UPDATED: 2025-12-26
STATUS: Production
PORT: 8006 (HTTP only)
MCP: stdio transport (stdin/stdout JSON-RPC)
UDP: None (no InterLock mesh)
PURPOSE: Multi-source web search with credibility scoring
CONFIG: ENV LOOKER_HTTP_PORT (default 8006)

---

ARCHITECTURE

SOURCES: Perplexity AI, NewsAPI, OpenAlex, DuckDuckGo
CREDIBILITY: Domain reputation, SSL, content quality, bias detection

TOOL: enhanced_search
INPUT: { query: string (required), sources?: ["perplexity"|"newsapi"|"openalex"|"duckduckgo"|"all"], maxResults?: 1-50 (default: 10), credibilityThreshold?: 0-1 (default: 0.3), options?: { sortBy: "relevance"|"credibility"|"date", filterDuplicates: boolean, enableCaching: boolean } }
OUTPUT: { results: [{ title, url, snippet, source, credibility_score }], total_found, sources_used }
USE: Multi-source web search with credibility filtering
EXAMPLE: enhanced_search({ query: "climate change research 2025", sources: ["perplexity", "openalex"], credibilityThreshold: 0.7 })

TOOL: validate_source
INPUT: { url: string (required), contentType?: "webpage"|"academic_paper"|"news_article"|"blog_post"|"social_media"|"government"|"other", verificationLevel?: "basic"|"standard"|"comprehensive"|"forensic" }
OUTPUT: { credibility_score: 0-1, domain_reputation, ssl_valid, bias_indicators, content_quality, verdict }
USE: Verify source credibility before citing
EXAMPLE: validate_source({ url: "https://example.com/article", verificationLevel: "comprehensive" })

TOOL: search_history
INPUT: { action: "list"|"search"|"export"|"delete"|"clear"|"stats" (required), timeRange?: { preset: "today"|"week"|"month"|"year"|"all" }, options?: { limit: number, sortBy: string } }
OUTPUT: { history: [], stats: { total_searches, sources_used } }
USE: Manage and review past searches
EXAMPLE: search_history({ action: "list", timeRange: { preset: "week" } })

TOOL: content_analysis
INPUT: { content: string (required, 1-50000 chars), analysisType?: "quality"|"relevance"|"sentiment"|"readability"|"comprehensive"|"bias"|"factual", context?: { referenceQuery: string, audience: "general"|"academic"|"technical"|"children" } }
OUTPUT: { scores: { quality, readability, bias }, sentiment, key_claims, fact_check_flags }
USE: Analyze content quality, bias, and factual accuracy
EXAMPLE: content_analysis({ content: "Article text here...", analysisType: "comprehensive" })

TOOL: cross_reference
INPUT: { primaryUrl: string (required), searchDepth?: "shallow"|"standard"|"deep", options?: { findBacklinks: boolean, findCitations: boolean, findRelatedContent: boolean, maxResults: number } }
OUTPUT: { backlinks: [], citations: [], related_content: [], authority_score }
USE: Find related content, backlinks, and citations for a URL
EXAMPLE: cross_reference({ primaryUrl: "https://example.com/research", searchDepth: "deep" })

---

SOURCES

- Perplexity AI: AI-powered search
- NewsAPI: News aggregation
- OpenAlex: Academic literature
- DuckDuckGo: Privacy-focused web search

CREDIBILITY: Domain reputation, SSL, content quality, bias detection

---

KEY FILES

SOURCE: /repo/looker-mcp/
INDEX: src/index.ts
ENGINES: src/engines/searchEngine.ts
SCORING: src/scoring/credibilityScorer.ts
SERVER: src/server/httpServer.ts
TYPES: src/types.ts
UTILS: src/utils/logger.ts

DEPENDENCIES: @modelcontextprotocol/sdk, uuid, express
