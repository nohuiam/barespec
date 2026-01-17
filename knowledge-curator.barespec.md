SERVER: knowledge-curator
VERSION: 1.4
UPDATED: 2026-01-17
STATUS: Production
PORT: 3017 (UDP/InterLock), 8017 (HTTP), 9017 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: Knowledge transformation with 70-90% token reduction while preserving 100% essential information
CONFIG: /repo/knowledge-curator/config/interlock.json

---

ARCHITECTURE

WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
DATABASE: SQLite with FTS5 (better-sqlite3)
COMPRESSION: SCOPE method (Semantic Chunking with Optimized Processing for Efficiency)
GLEC: Genesis (strategy), Leviticus (process), Exodus (implementation), Context (reference)
INTERLOCK: BaNano protocol, signals 0xB0-0xBF (CURATE_REQUEST, CURATE_COMPLETE)

---

HTTP REST API (port 8017)

ENDPOINT: GET /health
OUTPUT: { status: "healthy", server: string, version: string, uptime: { ms: number, human: string }, ports: object, timestamp: string }
USE: Health check and server status

ENDPOINT: GET /stats
OUTPUT: { success: boolean, timeRange: string, aggregated: { totalDocuments, avgCompressionRatio, totalTokensSaved, avgProcessingTimeMs }, recentDocuments: array }
USE: Compression statistics for last 24 hours

ENDPOINT: GET /api/v1/entries
PARAMS: ?limit=50&offset=0&glec=Genesis&subject=Architecture
OUTPUT: { success: boolean, count: number, entries: array }
USE: List PK entries with pagination and filtering

ENDPOINT: GET /api/v1/entries/:id
OUTPUT: { success: boolean, entry: PKEntry }
USE: Get specific PK entry by ID

ENDPOINT: GET /api/v1/search
PARAMS: ?q=query&limit=20
OUTPUT: { success: boolean, query: string, count: number, results: array }
USE: Full-text search via FTS5

ENDPOINT: GET /api/v1/graph/:id
PARAMS: ?depth=2&types=cites,extends
OUTPUT: { success: boolean, startNode: string, depth: number, nodes: array, edges: array }
USE: Query knowledge graph relationships

ENDPOINT: GET /api/tools
OUTPUT: { tools: [{ name, description, inputSchema }], count: number }
USE: List all MCP tools for bop-gateway integration

ENDPOINT: POST /api/tools/:toolName
INPUT: { arguments: object } or direct args object
OUTPUT: { success: boolean, result: object } or { success: false, error: string }
USE: Execute MCP tool via HTTP for bop-gateway integration

---

WEBSOCKET EVENTS (port 9017)

EVENT: connected
DIRECTION: server → client
DATA: { clientId: string, server: string, version: string, availableEvents: array }
USE: Sent on connection establishment

EVENT: curation_started
DIRECTION: server → client
DATA: { documentId: string, filePath: string, classification: string }
USE: Broadcast when curation begins

EVENT: curation_complete
DIRECTION: server → client
DATA: { documentId: string, title: string, compressionRatio: string, tokensSaved: number, processingTimeMs: number }
USE: Broadcast when curation finishes

EVENT: stats_update
DIRECTION: server → client
DATA: { totalDocuments: number, avgCompressionRatio: string, totalTokensSaved: number }
USE: Periodic statistics broadcast

EVENT: crossref_created
DIRECTION: server → client
DATA: { sourceId: string, targetId: string, referenceType: string }
USE: Broadcast when cross-reference created

CLIENT MESSAGE: subscribe
DATA: { type: "subscribe", events: string[] }
USE: Subscribe to specific events

CLIENT MESSAGE: ping
DATA: { type: "ping" }
RESPONSE: { type: "pong", data: { timestamp: number } }
USE: Keepalive check

---

TOOLS (6)

TOOL: curate_knowledge
INPUT: { filePath: string (required), classification: { glec: "Genesis"|"Leviticus"|"Exodus"|"Context" (required), subject: string (required), confidence: 0-1 (required), keywords: string[] (required) }, options?: { compressionLevel: "low"|"medium"|"high", preserveFormatting: boolean, generateCrossRefs: boolean } }
OUTPUT: { pk_entry: { id: string, compressed_content: string, token_count: number, compression_ratio: number }, nuggets_extracted: array, cross_refs: array }
USE: Transform classified document into optimized PK entry
EXAMPLE: curate_knowledge({ filePath: "/path/doc.md", classification: { glec: "Genesis", subject: "Architecture", confidence: 0.9, keywords: ["design", "system"] } })
NOTES: Compression ratio typically 70-90%. Higher compression = more aggressive summarization.

TOOL: analyze_nugget_patterns
INPUT: { nuggetIds: string[] (required), analysisType: "frequency"|"relationships"|"trends" (required) }
OUTPUT: { patterns: array, insights: array, recommendations: array }
USE: Analyze patterns in gold nuggets for insights
EXAMPLE: analyze_nugget_patterns({ nuggetIds: ["nug_001", "nug_002"], analysisType: "relationships" })
NOTES: "relationships" finds connections between nuggets. "trends" requires time-series data.

TOOL: update_search_index
INPUT: { pkId: string (required), content: string (required), keywords: string[] (required), metadata?: object }
OUTPUT: { indexed: boolean, index_id: string, searchable_terms: string[] }
USE: Update search index with a PK entry
EXAMPLE: update_search_index({ pkId: "pk_001", content: "Document content", keywords: ["key1", "key2"] })
NOTES: Called automatically by curate_knowledge. Use directly for manual index updates.

TOOL: build_cross_references
INPUT: { sourceId: string (required), targetIds: string[] (required), referenceType: "cites"|"extends"|"contradicts"|"supports" (required) }
OUTPUT: { references_created: number, graph_updated: boolean }
USE: Build typed relationships between documents
EXAMPLE: build_cross_references({ sourceId: "doc_001", targetIds: ["doc_002", "doc_003"], referenceType: "extends" })
NOTES: Reference types affect knowledge graph traversal. "contradicts" flags for human review.

TOOL: get_compression_stats
INPUT: { timeRange?: "hour"|"day"|"week"|"month" }
OUTPUT: { total_documents: number, avg_compression_ratio: number, tokens_saved: number, processing_time_avg: number }
USE: Get compression performance metrics
EXAMPLE: get_compression_stats({ timeRange: "week" })
NOTES: tokens_saved = original tokens - compressed tokens. Use for optimization tuning.

TOOL: query_knowledge_graph
INPUT: { startNode: string (required), depth: number (required, max: 5), relationshipTypes?: string[] }
OUTPUT: { nodes: array, edges: array, paths: array }
USE: Query document relationships in knowledge graph
EXAMPLE: query_knowledge_graph({ startNode: "doc_001", depth: 2, relationshipTypes: ["extends", "cites"] })
NOTES: Depth > 3 can return large graphs. Filter by relationshipTypes for focused queries.

---

KEY FILES

SOURCE: /repo/knowledge-curator/
INDEX: src/index.ts
TYPES: src/types.ts
DATABASE: src/database/schema.ts
COMPRESSION: src/compression/compressor.ts
CROSSREF: src/crossref/builder.ts
HTTP: src/http/server.ts
WEBSOCKET: src/websocket/server.ts
TESTS: tests/compression.test.ts, tests/database.test.ts
CONFIG: config/interlock.json

---

DEPENDENCIES

RUNTIME: @modelcontextprotocol/sdk, better-sqlite3, express, ws, cors, winston, lru-cache, uuid, zod
DEV: typescript, vitest, @types/node, @types/better-sqlite3, @types/express, @types/ws, @types/cors, @types/uuid
