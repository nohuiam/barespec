TOOLS: looker
VERSION: 2.0
UPDATED: 2025-12-25
COUNT: 5

---

TOOL: enhanced_search
INPUT: { query: string (required), sources?: ["perplexity"|"newsapi"|"openalex"|"duckduckgo"|"all"], maxResults?: 1-50 (default: 10), credibilityThreshold?: 0-1 (default: 0.3), options?: { sortBy: "relevance"|"credibility"|"date", filterDuplicates: boolean, enableCaching: boolean } }
OUTPUT: { results: [{ title: string, url: string, snippet: string, source: string, credibility_score: number }], total_found: number, sources_used: string[] }
USE: Multi-source web search with credibility filtering
EXAMPLE: enhanced_search({ query: "climate change research 2025", sources: ["perplexity", "openalex"], credibilityThreshold: 0.7 })
NOTES: Use "all" sources for comprehensive search. Higher credibilityThreshold = fewer but more reliable results.

---

TOOL: validate_source
INPUT: { url: string (required), contentType?: "webpage"|"academic_paper"|"news_article"|"blog_post"|"social_media"|"government"|"other", verificationLevel?: "basic"|"standard"|"comprehensive"|"forensic" }
OUTPUT: { credibility_score: 0-1, domain_reputation: object, ssl_valid: boolean, bias_indicators: string[], content_quality: object, verdict: string }
USE: Verify source credibility before citing
EXAMPLE: validate_source({ url: "https://example.com/article", verificationLevel: "comprehensive" })
NOTES: Use "forensic" for high-stakes verification. Returns bias indicators and quality metrics.

---

TOOL: search_history
INPUT: { action: "list"|"search"|"export"|"delete"|"clear"|"stats" (required), timeRange?: { preset: "today"|"week"|"month"|"year"|"all" }, options?: { limit: number, sortBy: string } }
OUTPUT: { history: array, stats: { total_searches: number, sources_used: object } }
USE: Manage and review past searches
EXAMPLE: search_history({ action: "list", timeRange: { preset: "week" } })
NOTES: "stats" action returns aggregate metrics. "clear" requires confirmation.

---

TOOL: content_analysis
INPUT: { content: string (required, 1-50000 chars), analysisType?: "quality"|"relevance"|"sentiment"|"readability"|"comprehensive"|"bias"|"factual", context?: { referenceQuery: string, audience: "general"|"academic"|"technical"|"children" } }
OUTPUT: { scores: { quality: number, readability: number, bias: number }, sentiment: object, key_claims: string[], fact_check_flags: string[] }
USE: Analyze content quality, bias, and factual accuracy
EXAMPLE: content_analysis({ content: "Article text here...", analysisType: "comprehensive" })
NOTES: "comprehensive" runs all analysis types. Fact check flags indicate claims needing verification.

---

TOOL: cross_reference
INPUT: { primaryUrl: string (required), searchDepth?: "shallow"|"standard"|"deep", options?: { findBacklinks: boolean, findCitations: boolean, findRelatedContent: boolean, maxResults: number } }
OUTPUT: { backlinks: array, citations: array, related_content: array, authority_score: number }
USE: Find related content, backlinks, and citations for a URL
EXAMPLE: cross_reference({ primaryUrl: "https://example.com/research", searchDepth: "deep" })
NOTES: "deep" search takes longer but finds more connections. Authority score based on citation network.

---

SOURCES: Perplexity AI, NewsAPI, OpenAlex, DuckDuckGo
CREDIBILITY: Domain reputation, SSL validation, content quality, bias detection
