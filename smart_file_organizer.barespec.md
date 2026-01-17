SERVER: smart-file-organizer
VERSION: 2.0.0
UPDATED: 2026-01-16
STATUS: Production
PORT: 3007 (UDP/InterLock), 8007 (HTTP), 9007 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: Heuristic GLEC file organization with rollback support
CONFIG: /repo/smart_file_organizer/config/interlock.json
DEPS: Redis 6379 (optional)

---

ARCHITECTURE

CLASSIFICATION: Heuristic GLEC (Genesis, Leviticus, Exodus, Context)
DATABASE: SQLite (better-sqlite3)
LAYERS: MCP stdio (3 tools), InterLock UDP mesh, HTTP REST, WebSocket events
ROLLBACK: Full operation history with undo capability

---

TOOLS (3)

TOOL: organize_files
INPUT: { sourcePath: string (required), options?: { dryRun?: boolean (default: false), validateAll?: boolean (default: true), batchSize?: number (default: 10), confidenceThreshold?: number (default: 0.3) } }
OUTPUT: { success: boolean, files_organized, files_skipped, movements: [] }
USE: Organize files using heuristic GLEC classification
EXAMPLE: organize_files({ sourcePath: "/Dropository", options: { dryRun: true } })
NOTES: Always use dryRun: true first. Creates movement records for rollback.

TOOL: get_organization_stats
INPUT: { timeRange?: "hour"|"day"|"week"|"month"|"all" (default: "day"), includeReviewQueue?: boolean (default: true), includeErrorAnalysis?: boolean (default: true) }
OUTPUT: { total_organized, by_category, review_queue, errors }
USE: Get organization statistics and metrics
EXAMPLE: get_organization_stats({ timeRange: "week" })

TOOL: rollback_operation
INPUT: { movementId?: string, batchId?: string, timeRange?: "hour"|"day"|"all", confirmDangerous?: boolean (default: false) }
OUTPUT: { success: boolean, files_restored, errors }
USE: Rollback file movements to restore original locations
EXAMPLE: rollback_operation({ movementId: "abc123" })
NOTES: Requires confirmDangerous: true for batch rollbacks

---

HTTP REST API (Port 8007)

GET  /health                    → { status, server, version, uptime, ports, database, timestamp }
GET  /api/tools                 → { tools: [], count: number } (Gateway integration)
POST /api/tools/:toolName       → { success: boolean, result: object } (Gateway integration)
GET  /stats                     → { success, stats, timestamp }
GET  /api/v1/movements          → List recent file movements
GET  /api/v1/movements/:id      → Get specific movement details
GET  /api/v1/review-queue       → Get files pending review
GET  /api/v1/rules              → Get classification rules

---

WEBSOCKET EVENTS (Port 9007)

S→C file_organized             → { path, category, confidence }
S→C batch_complete             → { total, success, failed }
S→C rollback_complete          → { movementId, restored }
C↔S ping/pong                  → Keepalive

---

INTERLOCK SIGNALS

Emits: FILE_ORGANIZED (0x70)
Accepts: ORGANIZE_REQUEST (0x71), HEARTBEAT (0x00)

---

KEY FILES

SOURCE: /repo/smart_file_organizer/
INDEX: src/server.js
TOOLS: src/tools/organize_files.js, src/tools/get_organization_stats.js, src/tools/rollback_operation.js
CLASSIFICATION: src/classifier/heuristic-classifier.js
DATABASE: src/database/client.js
HTTP: src/http/server.js
WEBSOCKET: src/websocket/server.js
INTERLOCK: src/interlock/socket.js
CONFIG: config/server.json, config/interlock.json

DEPENDENCIES: @modelcontextprotocol/sdk, better-sqlite3, express, ws, zod
