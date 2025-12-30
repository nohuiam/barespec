SERVER: bonzai-bloat-buster
VERSION: 3.2
UPDATED: 2025-12-30
STATUS: Production
PORT: 3008 (UDP/InterLock), 8008 (HTTP), 9008 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: Qdrant 6333 (vector DB - required)
PURPOSE: Document deduplication via vector similarity detection
CONFIG: /repo/bonzai-bloat-buster/config/interlock.json

---

ARCHITECTURE

WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
VECTOR DB: Qdrant (local)
EMBEDDINGS: all-MiniLM-L6-v2 (384 dimensions)
CONCURRENCY: p-queue with 10 parallel operations
INTERLOCK: BaNano protocol, signals 0x80-0x8F (DUPLICATE_SCAN, DUPLICATE_DETECTED, BLOAT_ANALYSIS)

---

TOOLS (6)

TOOL: analyze_directory
INPUT: { directoryPath: string (required), generatePlan?: boolean (default: true) }
OUTPUT: { files_analyzed, duplicates_found, consolidation_plan }
USE: Scan directory for duplicate/similar markdown documents
EXAMPLE: analyze_directory({ directoryPath: "/Users/macbook/Documents/inbox" })
NOTES: First step before any consolidation. Always generates plan unless disabled.

TOOL: process_documents
INPUT: { filePaths: string[] (required) }
OUTPUT: { processed_count, vectors_created }
USE: Add specific documents to vector database for future comparisons
EXAMPLE: process_documents({ filePaths: ["/path/doc1.md", "/path/doc2.md"] })
NOTES: Use when adding new documents to the index without full directory scan.

TOOL: compare_documents
INPUT: { filePathA: string (required), filePathB: string (required) }
OUTPUT: { semantic_similarity: 0-1, structural_similarity: 0-1, exact_similarity: 0-1, classification: "duplicate"|"near_duplicate"|"related"|"different" }
USE: Direct comparison of two specific documents
EXAMPLE: compare_documents({ filePathA: "/path/a.md", filePathB: "/path/b.md" })
NOTES: Returns three similarity scores. Classification based on semantic_similarity threshold.

TOOL: get_storage_stats
INPUT: {}
OUTPUT: { collection_info, vector_count, cache_stats }
USE: Check vector database health and statistics
EXAMPLE: get_storage_stats({})
NOTES: No input required. Use for monitoring Qdrant health.

TOOL: execute_consolidation_plan
INPUT: { planPath: string (required), dryRun?: boolean (default: true) }
OUTPUT: { actions_taken, files_merged, files_archived, backup_location }
USE: Execute consolidation plan with safety backups. Always use dryRun=true first.
EXAMPLE: execute_consolidation_plan({ planPath: "/path/plan.json", dryRun: true })
NOTES: ALWAYS use dryRun=true first. Creates backup before any destructive action.

TOOL: batch_compare_documents
INPUT: { newFiles: string[] (required), existingFiles?: string[], threshold?: number (default: 0.70), returnOnlyHighSimilarity?: boolean (default: true) }
OUTPUT: { comparisons: [{ file, matches: [{ path, similarity }] }] }
USE: High performance batch comparison. 50x faster than sequential.
EXAMPLE: batch_compare_documents({ newFiles: ["/path/new1.md", "/path/new2.md"], threshold: 0.80 })
NOTES: Use for bulk duplicate detection. Threshold 0.70 catches related content.

---

SIMILARITY THRESHOLDS

- >= 0.92: Exact duplicate
- 0.80-0.92: Near duplicate (same info, different wording)
- 0.70-0.80: Related content
- < 0.70: Different documents

---

HTTP REST API (port 8008)

ENDPOINT: GET /health
OUTPUT: { status: string, server: string, version: string, uptime: object, ports: object, qdrant: object, timestamp: string }
USE: Health check with Qdrant status

ENDPOINT: GET /stats
OUTPUT: { success: boolean, stats: { qdrant: object, cache: object }, timestamp: string }
USE: Storage and cache statistics

ENDPOINT: GET /api/v1/documents
OUTPUT: { success: boolean, count: number, documents: array, timestamp: string }
USE: List all indexed documents

ENDPOINT: GET /api/v1/documents/:identifier
OUTPUT: { success: boolean, document: object, timestamp: string }
USE: Get document by path or hash

ENDPOINT: GET /api/v1/collection
OUTPUT: { success: boolean, collection: object, timestamp: string }
USE: Qdrant collection info

ENDPOINT: GET /api/v1/cache
OUTPUT: { success: boolean, cache: object, timestamp: string }
USE: Embedding cache statistics

---

WEBSOCKET EVENTS (port 9008)

EVENT: connected
DIRECTION: server -> client
DATA: { clientId: string, server: string, version: string, qdrant: object, availableEvents: array }
USE: Sent on connection establishment

EVENT: document_processed
DIRECTION: server -> client
DATA: { filepath: string, hash: string, chunks: number, wordCount: number, timestamp: string }
USE: Broadcast when document is embedded

EVENT: comparison_complete
DIRECTION: server -> client
DATA: { fileA: string, fileB: string, matchType: string, semantic: number, structural: number, exact: number, timestamp: string }
USE: Broadcast when pair comparison completes

EVENT: duplicate_detected
DIRECTION: server -> client
DATA: { filepath: string, duplicateOf: string, similarity: number, matchType: string, timestamp: string }
USE: Broadcast when high-similarity match found

EVENT: plan_generated
DIRECTION: server -> client
DATA: { planId: string, actions: number, riskLevel: string, timestamp: string }
USE: Broadcast when consolidation plan ready

EVENT: consolidation_progress
DIRECTION: server -> client
DATA: { planId: string, step: number, total: number, currentAction: string, timestamp: string }
USE: Real-time execution progress

EVENT: stats_update
DIRECTION: server -> client
DATA: { qdrant: object, cache: object, timestamp: string }
USE: Periodic stats broadcast (30s)

CLIENT MESSAGE: subscribe
DATA: { type: "subscribe", events: string[] }
USE: Subscribe to specific events

CLIENT MESSAGE: ping
DATA: { type: "ping" }
RESPONSE: { type: "pong", data: { timestamp: number } }
USE: Keepalive check

---

KEY FILES

SOURCE: /repo/bonzai-bloat-buster/
INDEX: src/index.ts
TYPES: src/types.ts
HTTP: src/http/server.ts
WEBSOCKET: src/websocket/server.ts
EMBEDDING: src/embedding/embeddingPipeline.ts, src/embedding/vectorStorage.ts
DETECTION: src/detection/similarityDetector.ts, src/detection/obsolescenceDetector.ts
ANALYSIS: src/analysis/reportGenerator.ts, src/analysis/consolidationPlanner.ts, src/analysis/consolidationExecutor.ts
PROCESSING: src/processing/documentChunker.ts
TESTS: tests/http.test.ts, tests/websocket.test.ts

DEPENDENCIES: @modelcontextprotocol/sdk, @xenova/transformers, @qdrant/js-client-rest, express, ws, cors, better-sqlite3, winston, p-queue
