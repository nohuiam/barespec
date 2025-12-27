SERVER: smart-file-organizer
VERSION: 2.1
UPDATED: 2025-12-27
STATUS: Production
PORT: 3007 (UDP/InterLock), 8007 (HTTP), 9007 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: Redis 6379 (vault)
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

LAYERS

1. MCP stdio (3 tools) - Primary interface
2. InterLock UDP mesh (port 3007) - Peer communication
3. HTTP REST API (port 8007) - External integrations
4. WebSocket real-time (port 9007) - Live updates

---

KEY FILES

SOURCE: /repo/smart_file_organizer/
INDEX: src/server.js
TOOLS: src/tools/organize_files.js, src/tools/get_organization_stats.js, src/tools/rollback_operation.js
SERVICES: src/services/classification.js, src/services/discovery.js, src/services/movement.js, src/services/analysis.js, src/services/consolidation.js
INTERLOCK: src/interlock/socket.js, src/interlock/vault.js, src/interlock/tumbler.js, src/interlock/handlers.js
DATABASE: src/database/init.js
CONFIG: config/tumbler.json

DEPENDENCIES: @modelcontextprotocol/sdk, better-sqlite3, redis
