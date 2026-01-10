SERVER: niws-analysis
VERSION: 1.0
UPDATED: 2026-01-10
STATUS: Production
PORTS: UDP 3034 | HTTP 8034 | WS 9034
MCP: stdio transport (stdin/stdout JSON-RPC)
INTERLOCK: Enabled (peer mesh communication)
DEPS: Claude API (for LLM analysis), niws-intake (article data)
PURPOSE: Bias analysis pipeline - article analysis, framing comparison, neutral language
CONFIG: ENV NIWS_ANALYSIS_PORT (default 8034), ANTHROPIC_API_KEY

---

ARCHITECTURE

FUNCTION: Second stage of NIWS pipeline - bias analysis and framing detection
FLOW: Articles from intake -> Bias Analysis -> Framing Comparison -> Neutral Alternatives
STORAGE: SQLite (analysis.db)
DATABASES:
  - analysisDatabase: Article analyses, comparisons, lexicon entries
ANALYSIS_TYPES: bias | framing | neutral | comprehensive
KEY_FEATURE: Observer-only analysis - documents WITHOUT judging accuracy

---

TOOL CATEGORIES

ANALYSIS (5 tools):
- analyze_article: Analyze single article for bias patterns (emphasis, omissions, sources, framing)
- analyze_bias_language: Check text for loaded language, suggest alternatives
- compare_coverage: Compare how multiple outlets cover same story across political spectrum
- get_framing_differences: Extract different outlet framings of same concept
- get_neutral_alternative: Get neutral alternatives for loaded terms (lexicon or LLM)

VALIDATION (1 tool):
- validate_analysis_text: Check text for bias violations before output

RETRIEVAL (5 tools):
- get_analysis_by_id: Retrieve analysis by ID
- get_analysis_by_story: Get all analyses for story plus comparative analysis
- get_comparative_analysis: Get comparison by ID or story ID
- list_pending_analyses: List analyses awaiting processing
- retry_failed_analysis: Retry a failed analysis

TOTAL: 11 tools

---

KEY CONCEPTS

BIAS_ANALYSIS: Documents emphasis, omissions, sources, framing without judgment
FRAMING_COMPARISON: How outlets frame same concept with different language
LOADED_LANGUAGE: Words with emotional charge that could influence perception
LEXICON: Database of loaded terms with neutral alternatives and severity scores
POLITICAL_LEAN: left | left-center | center | right-center | right
OUTLET_FRAMING: {outlet_name, outlet_lean, phrase}
NEUTRALITY_SCORE: 0-1 score from output validation
VALIDATION_RESULT: {valid, violations, warnings, neutralityScore}

---

ANALYSIS TYPES

TYPE: bias
DESC: Identify loaded language and emotional framing
OUTPUT: loaded_terms, emotional_appeals, source_selection

TYPE: framing
DESC: Document how story is positioned/contextualized
OUTPUT: headline_framing, emphasis_patterns, omissions

TYPE: neutral
DESC: Generate neutral language alternatives
OUTPUT: original_phrases, neutral_alternatives, severity_scores

TYPE: comprehensive
DESC: Full analysis combining all types
OUTPUT: All of the above plus overall_assessment

---

DATABASE SCHEMA

TABLE: analyses
COLUMNS: id, article_id, story_id, analysis_type, status, result, error_message, created_at, completed_at

TABLE: comparisons
COLUMNS: id, story_id, article_ids, status, framing_differences, overall_assessment, error_message, created_at, completed_at

TABLE: lexicon
COLUMNS: id, word, category, lean, severity, alternatives, created_at

---

HTTP ENDPOINTS

GET /api/health - Health check (supports deep=true query param)
GET /metrics - Prometheus metrics (optional auth via METRICS_AUTH_TOKEN)
GET /api/analyses - List analyses with filters (articleId, type, status, limit, offset)
GET /api/analyses/:id - Get analysis by ID
GET /api/analyses/story/:storyId - Get all analyses for story
POST /api/analyze - Run analysis (rate limited: 10/min)
POST /api/compare - Run comparison (rate limited: 10/min)
GET /api/lexicon - Get loaded language lexicon (optional category filter)
GET /api/lexicon/:word - Get lexicon entry for word
POST /api/lexicon - Add lexicon entry
GET /api/stats - Database statistics

---

WEBSOCKET EVENTS

EVENT: analysis:started - Analysis begun for article
EVENT: analysis:complete - Analysis finished successfully
EVENT: analysis:failed - Analysis failed with error
EVENT: comparison:started - Story comparison begun
EVENT: comparison:complete - Comparison finished

---

INTERLOCK SIGNALS

EMIT:
- server:ready - Server operational
- server:shutdown - Server shutting down
- analysis:started - Analysis begun
- analysis:complete - Analysis finished
- analysis:failed - Analysis error
- comparison:started - Comparison begun
- comparison:complete - Comparison finished

RECEIVE:
- article:new - New article from intake (trigger analysis)
- story:clustered - Story cluster formed (trigger comparison)
- analysis:request - External analysis request

PEERS: niws-intake (3033), niws-production (3035), niws-delivery (3036)

---

RATE LIMITING

GENERAL: 100 requests/minute (all endpoints)
ANALYSIS: 10 requests/minute (POST /api/analyze, POST /api/compare)
REASON: Protects Claude API from excessive calls
STORE: In-memory (use REDIS_URL for multi-instance)

---

KEY FILES

SOURCE: /repo/niws-analysis/
INDEX: src/index.ts
HTTP: src/http/server.ts
WS: src/websocket/server.ts
INTERLOCK: src/interlock/
TOOLS: src/tools/index.ts
DATABASE: src/database/analysisDatabase.ts
SERVICES:
  - src/services/biasAnalyzer.ts
  - src/services/outputValidator.ts
  - src/services/healthChecker.ts
LOGGING: src/logging/index.ts
METRICS: src/metrics/index.ts
CONFIG: config/interlock.json
DATA: data/analysis.db
