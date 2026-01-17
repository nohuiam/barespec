SERVER: safe-batch-processor
VERSION: 1.0.0
UPDATED: 2026-01-08
STATUS: Tested (10 HTTP tests passed)
PORT: 3022 (UDP/InterLock), 8022 (HTTP), 9022 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk, cors, express-rate-limit
PURPOSE: Safe batch file operations with validation, backup, and rollback capability
CONFIG: /repo/safe-batch-processor/config/interlock.json
TESTS: 10 HTTP tests (health, 404, CORS x6, rate limit x2)

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3)
WORKFLOW: Validate → Backup → Execute → Track (auto-rollback on failure threshold)
SAFETY: Max 50 operations per batch, auto-rollback at 20% failure rate
BACKUP_RETENTION: 30 days

---

TOOLS (4)

TOOL: execute_batch
INPUT: { operations: array (required), validation?: boolean, createBackup?: boolean }
OUTPUT: { batch_id: string, success_count: number, failure_count: number, rollback_available: boolean, backup_id: string, errors: [] }
USE: Execute batch operation with safety features (validation + backup)
EXAMPLE: execute_batch({ operations: [{ type: "move", source: "/a.txt", target: "/b.txt" }], createBackup: true })
NOTES: Operations: move, copy, delete, rename. Auto-rollback triggers at 20% failure rate.

TOOL: validate_batch
INPUT: { operations: array (required) }
OUTPUT: { valid: boolean, issues: [], estimated_time_seconds: number, total_size_bytes: number }
USE: Pre-validate operations before execution
EXAMPLE: validate_batch({ operations: [{ type: "delete", target: "/temp/*.log" }] })
NOTES: Checks path safety, permissions, file existence, overwrite warnings

TOOL: rollback_batch
INPUT: { batch_id: string (required), backup_id?: string }
OUTPUT: { success: boolean, restored: number, failed: number }
USE: Rollback batch operation to previous state using backup
EXAMPLE: rollback_batch({ batch_id: "batch-123" })
NOTES: Requires backup to have been created during execute_batch

TOOL: get_batch_status
INPUT: { batch_id?: string, limit?: number }
OUTPUT: { batch?: object, history?: [] }
USE: Get status of specific batch or list batch history
EXAMPLE: get_batch_status({ limit: 10 })
NOTES: Returns detailed progress for in-flight batches, history for completed

---

SAFETY LIMITS

| Limit | Value |
|-------|-------|
| Max operations per batch | 50 (hard limit) |
| Max concurrent operations | 5 (default) |
| Backup retention | 30 days |
| Auto-rollback threshold | 20% failure rate |

VALIDATION CHECKS:
- Path safety (protected paths blocked)
- Permission verification
- File existence validation
- Target overwrite warnings

---

LAYERS

1. MCP stdio (4 tools) - Primary interface
2. InterLock UDP mesh (port 3022) - Peer communication
3. HTTP REST API (port 8022) - External integrations
4. WebSocket real-time (port 9022) - Live updates

---

HTTP REST API

GET  /health               → { status, active_batches, backup_count }
POST /api/batch            → { batch_id, success_count, failure_count, ... }
POST /api/validate         → { valid, issues }
POST /api/rollback         → { success, restored, failed }
GET  /api/batches          → { batches[], total }
GET  /api/batches/:id      → { batch details }
GET  /api/batches/:id/items → { items[], progress }

---

WEBSOCKET EVENTS

S→C batch_started          → Batch execution started
S→C batch_progress         → Progress update (item completed)
S→C batch_complete         → Batch finished successfully
S→C batch_failed           → Batch failed (threshold exceeded)
S→C rollback_started       → Rollback initiated
S→C rollback_complete      → Rollback finished
C↔S ping/pong              → Keepalive

---

INTERLOCK SIGNALS

Emits: HEARTBEAT (0x01), BATCH_STARTED (0x30), BATCH_COMPLETE (0x31), BATCH_FAILED (0x32), ROLLBACK_STARTED (0x33), ROLLBACK_COMPLETE (0x34)
Accepts: HEARTBEAT, DISCOVERY, SHUTDOWN, HEALTH_CHECK, HEALTH_RESPONSE

PEERS: consolidation-engine (3032), project-context (3016), snapshot (3003), smart-file-organizer (3007)

---

DATABASE SCHEMA

TABLE: batch_operations
COLUMNS: id, batch_id, status, operation_type, total_count, success_count, failure_count, created_at, completed_at, backup_id
INDEXES: idx_batch_id, idx_batch_status, idx_batch_date

TABLE: batch_items
COLUMNS: id, batch_id, item_index, operation, source_path, target_path, status, error_message, processed_at
INDEXES: idx_items_batch, idx_items_status

TABLE: batch_backups
COLUMNS: id, backup_id, batch_id, backup_path, created_at, expires_at, restored
INDEXES: idx_backups_batch, idx_backups_expires

---

KEY FILES

SOURCE: /repo/safe-batch-processor/
INDEX: src/index.ts
CORE: src/core/ (backup.ts, executor.ts, rollback.ts, validator.ts)
INTERLOCK: src/interlock/socket.ts
HTTP: src/http/server.ts
WEBSOCKET: src/websocket/server.ts
DATABASE: data/batch-processor.db
BACKUPS: data/backups/
CONFIG: config/interlock.json

DEPENDENCIES: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk

---

SECURITY

CORS: Origin whitelist (localhost:5173, 127.0.0.1:5173, localhost:3099, localhost:8022)
RATE_LIMITS:
- General: 100 requests/minute
- Batch/Rollback: 20 requests/minute (conservative for file ops)
HEADERS: RateLimit-Limit, RateLimit-Remaining, RateLimit-Reset (draft-7)
AUDIT: Linus audit completed 2026-01-08 (Instance 8)

---

NOTES

1. Always validate before destructive operations
2. Backup is created automatically unless explicitly disabled
3. Auto-rollback triggers at 20% failure rate to prevent cascading damage
4. 30-day backup retention - old backups auto-purged
5. Protected paths (system directories) are blocked at validation
6. Max 50 operations per batch to ensure manageability
