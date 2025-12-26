SERVER: smart-file-organizer
VERSION: 2.0
UPDATED: 2025-12-26
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
LAYERS: MCP stdio (3 tools), InterLock UDP mesh (optional)

---

TOOLS (3)

TOOL: organize_files
INPUT: { sourcePath: string (required), options?: { dryRun: boolean (default: false), validateAll: boolean (default: true), batchSize: number (default: 10), confidenceThreshold: 0-1 (default: 0.3), classificationMode: "heuristic"|"catasorter"|"both" (default: "heuristic"), errorAbortThreshold: 0-1 (default: 0.2) } }
OUTPUT: { operation_id, files_discovered, files_organized, files_skipped, errors: [], movements: [{ from, to, classification }] }
USE: Organize files using heuristic GLEC classification with validation and batch processing
EXAMPLE: organize_files({ sourcePath: "/Dropository", options: { dryRun: true } })

TOOL: get_organization_stats
INPUT: { timeRange?: "hour"|"day"|"week"|"month"|"all" (default: "day"), includeReviewQueue?: boolean (default: true), includeErrorAnalysis?: boolean (default: true) }
OUTPUT: { total_operations, files_organized, success_rate, by_classification: {}, recent_movements?: [] }
USE: Get organization statistics and metrics
EXAMPLE: get_organization_stats({ timeRange: "week" })

TOOL: rollback_operation
INPUT: { movementId?: string, batchId?: string, timeRange?: "hour"|"day"|"all", confirmDangerous?: boolean (default: false) }
OUTPUT: { success: boolean, files_restored, errors: [] }
USE: Rollback file movements to restore files to original locations
EXAMPLE: rollback_operation({ movementId: "mov_123", confirmDangerous: true })

---

SERVICES

- FileDiscoveryService: Find files to organize
- HeuristicClassifier: Fast local classification
- FileMovementService: Safe file moving with tracking
- AnalysisService: Content analysis
- ConsolidationService: Duplicate handling

---

INTERLOCK SIGNALS

Emits: FILE_DISCOVERED (0x10)
Accepts: DOCK_REQUEST, HEARTBEAT, FILE_INDEXED

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
