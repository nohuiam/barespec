SERVER: catasorter
VERSION: 2.1.1
UPDATED: 2026-01-17
STATUS: Production
PORT: 3005 (UDP/InterLock), 8005 (HTTP), 9005 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: GLEC document classification and knowledge organization
CONFIG: /repo/Catasorter/config/interlock.json

v2.1 CHANGES: Rate limiting (token bucket + circuit breaker), parallel batch processing, web classification tools

---

ARCHITECTURE

CLASSIFICATION: GLEC framework (Genesis, Leviticus, Exodus, Context)
DEWEY: Hierarchical ID system for document tracking
NUGGETS: Gold nugget extraction for key insights
DATABASE: SQLite (better-sqlite3)
LAYERS: MCP stdio (12 tools), InterLock UDP mesh, HTTP REST, WebSocket events
RATE-LIMITING: Token bucket + circuit breaker per API (Claude, Perplexity, Ollama)
RETRY: Exponential backoff with jitter, configurable max retries

---

TOOLS (12) - 6 file classification + 6 web classification

TOOL: classify_document
INPUT: { file_path: string (required), extract_nuggets?: boolean (default: true), user_hint?: string }
OUTPUT: { classification: { primary_category, subject, glec: { genesis, leviticus, exodus, context }, confidence, importance, complexity }, dewey_id, gold_nuggets: [], metadata_filename }
USE: Classify a single document using GLEC framework with optional nugget extraction
EXAMPLE: classify_document({ file_path: "/Dropository/new-doc.md", extract_nuggets: true })
NOTES: GLEC percentages sum to 100. user_hint biases classification toward expected category.

TOOL: get_classification
INPUT: { file_path: string (required) }
OUTPUT: { file_path, file_name, classified_at, glec: { genesis, leviticus, exodus, context }, subject, dewey_id, confidence, importance, complexity, llm_method }
USE: Get classification for a previously classified document
EXAMPLE: get_classification({ file_path: "/Classified/Genesis/Architecture/doc.md" })
NOTES: Returns null if file not in classification database.

TOOL: get_dewey_stats
INPUT: {}
OUTPUT: { total_documents, counters: { Genesis: {}, Leviticus: {}, ... }, next_ids: {} }
USE: Get Dewey ID statistics and counters
EXAMPLE: get_dewey_stats({})
NOTES: next_ids shows what ID will be assigned next for each category/subject combination.

TOOL: get_workflow_metrics
INPUT: { include_details?: boolean (default: false) }
OUTPUT: { total_classifications, by_category: {}, by_subject: {}, by_method: {}, average_confidence, gold_nuggets: { total, in_project_knowledge, pending_review } }
USE: Get classification workflow metrics and statistics
EXAMPLE: get_workflow_metrics({ include_details: true })
NOTES: by_method shows LLM vs heuristic classification breakdown.

TOOL: organize_document
INPUT: { file_path: string (required), user_hint?: string, dry_run?: boolean (default: false) }
OUTPUT: { success: boolean, operation, classification: {}, original_path, new_path, folder_created, file_moved }
USE: Classify and organize document to final location in Classified folder
EXAMPLE: organize_document({ file_path: "/Dropository/doc.md", dry_run: true })
NOTES: Creates destination folder if needed. ALWAYS dry_run first.

TOOL: batch_process
INPUT: { max_files?: number (default: 10), dry_run?: boolean (default: false), extract_nuggets?: boolean (default: true), parallel_limit?: number (default: 5, max: 10) }
OUTPUT: { success: boolean, operation, files_discovered, files_processed, files_failed, parallel_limit, results: [] }
USE: Batch process documents from Dropository with parallel execution
EXAMPLE: batch_process({ max_files: 25, parallel_limit: 5, dry_run: true })
NOTES: Processes markdown and text files in parallel batches. Rate limiting prevents API overload.

---

WEB CLASSIFICATION TOOLS (6)

TOOL: classify_url
INPUT: { url: string (required), extract_nuggets?: boolean (default: true), mode?: string (default: "bop") }
OUTPUT: { url, domain, web_dewey_id, classification: { primary_category, subject, glec, confidence }, credibility, reliability, action_category, gold_nuggets: [], implementation_notes, pricing_info }
USE: Classify a web URL and extract nuggets
EXAMPLE: classify_url({ url: "https://arxiv.org/abs/2405.00732", mode: "bop" })
NOTES: Modes: "bop" (Box of Prompts focus), "niws" (news analysis with Christ-Oh-Meter), "default"

TOOL: get_web_classification
INPUT: { url: string (required) }
OUTPUT: { url, domain, web_dewey_id, classification, credibility, classified_at }
USE: Get classification for a previously classified URL
EXAMPLE: get_web_classification({ url: "https://example.com/article" })
NOTES: Returns null if URL not in database.

TOOL: get_domain_info
INPUT: { domain: string (required) }
OUTPUT: { domain, total_urls, classifications: [], average_credibility }
USE: Get all classifications for a specific domain
EXAMPLE: get_domain_info({ domain: "arxiv.org" })
NOTES: Useful for domain-level analysis.

TOOL: get_web_stats
INPUT: {}
OUTPUT: { total_urls, by_domain: {}, by_category: {}, average_credibility }
USE: Get web classification statistics
EXAMPLE: get_web_stats({})
NOTES: Overview of all web classifications.

TOOL: batch_classify_urls
INPUT: { urls: string[] (required), extract_nuggets?: boolean (default: true), mode?: string (default: "bop") }
OUTPUT: { success: boolean, total, processed, failed, results: [] }
USE: Batch classify multiple URLs
EXAMPLE: batch_classify_urls({ urls: ["https://a.com", "https://b.com"], mode: "bop" })
NOTES: Processes URLs with rate limiting. Results include per-URL success/error.

TOOL: discover_urls
INPUT: { source: string (required), max_urls?: number (default: 10) }
OUTPUT: { source, urls_discovered, urls: [] }
USE: Discover URLs from a source (file, webpage, or text)
EXAMPLE: discover_urls({ source: "/path/to/urls.txt", max_urls: 50 })
NOTES: Extracts URLs from markdown links, plain text, or HTML.

---

GLEC FRAMEWORK

- Genesis: Strategy, vision, architecture, concepts
- Leviticus: Process, workflow, procedures, coordination
- Exodus: Implementation, code, execution, technical
- Context: Reference, documentation, external sources

DEWEY FORMAT: {G|L|E|C}_{Subject}_{###} (e.g., G_Architecture_001)

---

HTTP REST API (Port 8005)

GET  /health                    → { status, server, version, uptime, database, timestamp }
GET  /api/tools                 → { tools: [], count: number } (Gateway integration)
POST /api/tools/:toolName       → { success: boolean, result: object } (Gateway integration)
GET  /stats                     → Classification statistics
GET  /api/classifications       → List classifications
POST /api/classify              → Classify document

---

INTERLOCK SIGNALS

Emits: CLASSIFICATION_COMPLETE (0x51)
Accepts: DOCK_REQUEST (0x01), HEARTBEAT (0x04), FILE_INDEXED (0x11), CLASSIFICATION_REQUEST (0x50)

---

KEY FILES

SOURCE: /repo/Catasorter/
INDEX: src/index.js
TOOLS: src/tools/index.js, src/tools/classify-document.js, src/tools/web-tools.js
CLASSIFICATION: src/classification/glec-classifier.js, src/classification/web-classifier.js
NAMING: src/naming/dewey-generator.js, src/naming/filename-builder.js
PK-INTEGRATION: src/pk-integration/nugget-extractor.js
COGNITIVE: src/cognitive/tenets-integration.js (Christ-Oh-Meter for NIWS mode)
INTERLOCK: src/interlock/socket.js, src/interlock/tumbler.js
HTTP: src/http/server.js
WEBSOCKET: src/websocket/server.js
DATABASE: src/database/client.js
CONFIG: config/server.json, config/interlock.json

UTILITIES (v2.1):
- src/utils/rate-limiter.js: Token bucket + circuit breaker (CLOSED/OPEN/HALF_OPEN states)
- src/utils/api-limiters.js: Per-API instances (claude, perplexity, ollama, tenets, experience)
- src/utils/retry.js: Exponential backoff with jitter, rate limiter integration
- src/utils/logger.js: Structured logging with JSON output
- src/constants/nugget-types.js: Single source of truth for nugget types

SCRIPTS:
- scripts/batch-extract.sh: Parallel URL extraction (xargs -P)
  Usage: echo "url1\nurl2" | PARALLEL=5 MODE=bop ./batch-extract.sh
  Env: PARALLEL (default: 5), SERVER (default: http://localhost:8005), MODE (bop/niws/default)

MIGRATIONS:
- migrations/004_unify_nugget_types.sql: Unified nugget types (added reference, tutorial)

DEPENDENCIES: @modelcontextprotocol/sdk, @anthropic-ai/sdk, better-sqlite3, zod, zod-to-json-schema, dotenv, express, ws, axios

---

BUGFIXES

v2.1.1 (2026-01-17):
- Fixed /api/tools endpoint returning empty tool names (BUG-001)
- Added zod-to-json-schema for proper JSON Schema conversion in gateway /api/tools response
- Object.values() → Object.entries() to preserve tool names in HTTP handler
