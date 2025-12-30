SERVER: smart-file-organizer
VERSION: 2.2
UPDATED: 2025-12-30
STATUS: Production
PORT: 3007 (UDP/InterLock), 8007 (HTTP), 9007 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: Redis 6379 (vault - optional)
PURPOSE: Heuristic file classification and organization with rollback support
CONFIG: /repo/smart_file_organizer/src/server.js

---

ARCHITECTURE

CLASSIFICATION: Heuristic (fast, local) or CataSORTER integration
SAFETY: Dry run option, full rollback support, operation tracking
INTERLOCK: UDP mesh (graceful degradation - server continues if InterLock fails), SharedMemoryVault (Redis), Tumbler signal filtering
DATABASE: SQLite (better-sqlite3), __dirname-based paths for Claude Desktop compatibility

---

TOOLS (3)

TOOL: organize_files
INPUT: { sourcePath: string (required), options?: { dryRun: boolean (default: false), validateAll: boolean (default: true), batchSize: number (default: 10), confidenceThreshold: 0-1 (default: 0.3), classificationMode: "heuristic"|"catasorter"|"both" (default: "heuristic"), errorAbortThreshold: 0-1 (default: 0.2) } }
OUTPUT: { operation_id: string, files_discovered: number, files_organized: number, files_skipped: number, errors: array, movements: [{ from: string, to: string, classification: object }] }
USE: Organize files using heuristic GLEC classification with validation and batch processing
EXAMPLE: organize_files({ sourcePath: "/Dropository", options: { dryRun: true } })
NOTES: Use dryRun first. "heuristic" is fast/local, "catasorter" uses external MCP, "both" combines.

TOOL: get_organization_stats
INPUT: { timeRange?: "hour"|"day"|"week"|"month"|"all" (default: "day"), includeReviewQueue?: boolean (default: true), includeErrorAnalysis?: boolean (default: true) }
OUTPUT: { total_operations: number, files_organized: number, success_rate: number, by_classification: object, recent_movements?: array }
USE: Get organization statistics and metrics
EXAMPLE: get_organization_stats({ timeRange: "week" })
NOTES: includeReviewQueue shows files needing manual review.

TOOL: rollback_operation
INPUT: { movementId?: string, batchId?: string, timeRange?: "hour"|"day"|"all", confirmDangerous?: boolean (default: false) }
OUTPUT: { success: boolean, files_restored: number, errors: array }
USE: Rollback file movements to restore files to original locations
EXAMPLE: rollback_operation({ movementId: "mov_123", confirmDangerous: true })
NOTES: Supports single file, batch, or time-range rollbacks. Requires confirmDangerous for dangerous ops.

---

CLASSIFICATION MODES

- heuristic: Fast, local, uses file patterns and content heuristics
- catasorter: Uses CataSORTER MCP for GLEC classification
- both: Runs heuristic first, uses catasorter for validation

---

SERVICES

- FileDiscoveryService: Watches /Dropository for new files
- HeuristicClassifier: Pattern-based classification
- FileMovementService: Safe file moves with audit trail
- AnalysisService: Content analysis
- ConsolidationService: Duplicate handling

---

INTERLOCK SIGNALS

Emits: FILE_DISCOVERED (0x10)
Accepts: DOCK_REQUEST, HEARTBEAT, FILE_INDEXED

---

HTTP REST API (port 8007)

ENDPOINT: GET /health
OUTPUT: { status: string, server: string, version: string, uptime: object, ports: object, database: string, timestamp: string }
USE: Health check with database status

ENDPOINT: GET /stats
OUTPUT: { success: boolean, stats: object, timestamp: string }
USE: Organization statistics

ENDPOINT: GET /api/v1/movements
OUTPUT: { success: boolean, count: number, movements: array, pagination: object, timestamp: string }
USE: Query recent file movements with pagination

ENDPOINT: GET /api/v1/movements/:movementId
OUTPUT: { success: boolean, movement: object, timestamp: string }
USE: Get specific movement details

ENDPOINT: GET /api/v1/review-queue
OUTPUT: { success: boolean, count: number, queue: array, timestamp: string }
USE: Get files pending manual review

ENDPOINT: GET /api/v1/rules
OUTPUT: { success: boolean, count: number, rules: array, timestamp: string }
USE: Get classification rules

---

WEBSOCKET EVENTS (port 9007)

EVENT: connected
DIRECTION: server -> client
DATA: { clientId: string, server: string, version: string, database: string, availableEvents: array }
USE: Sent on connection establishment

EVENT: file_discovered
DIRECTION: server -> client
DATA: { filepath: string, size: number, timestamp: string }
USE: Broadcast when new file discovered

EVENT: file_classified
DIRECTION: server -> client
DATA: { filepath: string, category: string, confidence: number, timestamp: string }
USE: Broadcast when file classified

EVENT: movement_started
DIRECTION: server -> client
DATA: { movementId: string, source: string, destination: string, timestamp: string }
USE: Broadcast when file movement begins

EVENT: movement_completed
DIRECTION: server -> client
DATA: { movementId: string, verified: boolean, timestamp: string }
USE: Broadcast when file movement completes

EVENT: movement_failed
DIRECTION: server -> client
DATA: { movementId: string, error: string, timestamp: string }
USE: Broadcast when file movement fails

EVENT: validation_request
DIRECTION: server -> client
DATA: { operationId: string, filepath: string, timestamp: string }
USE: Broadcast when file needs validation

EVENT: review_queue_updated
DIRECTION: server -> client
DATA: { queueId: string, filepath: string, timestamp: string }
USE: Broadcast when review queue changes

EVENT: stats_update
DIRECTION: server -> client
DATA: { stats: object, timestamp: string }
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

SOURCE: /repo/smart_file_organizer/
INDEX: src/server.js
TOOLS: src/tools/organize_files.js, src/tools/get_organization_stats.js, src/tools/rollback_operation.js
HTTP: src/http/server.js
WEBSOCKET: src/websocket/server.js
SERVICES: src/services/classification.js, src/services/discovery.js, src/services/movement.js, src/services/analysis.js, src/services/consolidation.js
INTERLOCK: src/interlock/socket.js, src/interlock/vault.js, src/interlock/tumbler.js, src/interlock/handlers.js
DATABASE: src/database/init.js
CONFIG: config/tumbler.json
TESTS: tests/http.test.js, tests/websocket.test.js

DEPENDENCIES: @modelcontextprotocol/sdk, better-sqlite3, ioredis, express, ws, cors, chokidar, winston, zod
