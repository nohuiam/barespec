SERVER: pk-manager
VERSION: 1.3
UPDATED: 2025-12-30
STATUS: Production
PORT: 3018 (UDP/InterLock), 8018 (HTTP), 9018 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: Project Knowledge rebuild orchestration with 0% data loss guarantee
CONFIG: /repo/pk-manager/config/interlock.json

---

ARCHITECTURE

WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
DATABASE: SQLite (better-sqlite3)
PIPELINE: 7-phase rebuild workflow
SAFETY: Automatic rollback on critical verification failure
INTERLOCK: BaNano protocol, signals 0xD0-0xDF (REBUILD_REQUEST, REBUILD_COMPLETE)

---

HTTP REST API (port 8018)

ENDPOINT: GET /health
OUTPUT: { status: "healthy", server: string, version: string, uptime: { ms: number, human: string }, activeRebuild: string|null, ports: object, timestamp: string }
USE: Health check and server status

ENDPOINT: GET /stats
OUTPUT: { success: boolean, stats: { rebuildsTotal: number, rebuildsSuccess: number, rebuildsFailed: number, rebuildsRolledBack: number, successRate: string, avgDurationMs: number, avgDurationHuman: string }, activeRebuild: string|null, timestamp: string }
USE: Rebuild statistics and success rates

ENDPOINT: GET /api/v1/rebuilds
PARAMS: ?limit=20&status=success|failed|rolled_back
OUTPUT: { success: boolean, count: number, limit: number, activeRebuild: string|null, rebuilds: array }
USE: List rebuild operations with pagination and filtering

ENDPOINT: GET /api/v1/rebuilds/:id
OUTPUT: { success: boolean, rebuild: { id, status, sourcePath, backupPath, documentsTotal, documentsProcessed, documentsSucceeded, documentsFailed, phases, verificationResults, startedAt, completedAt, duration, error }, backups: array }
USE: Get specific rebuild with phases and backups

ENDPOINT: GET /api/v1/backups
PARAMS: ?rebuildId=xxx OR ?latest=true
OUTPUT: { success: boolean, count: number, backups: array } OR { success: boolean, backup: object|null }
USE: List backups for rebuild or get latest backup

---

WEBSOCKET EVENTS (port 9018)

EVENT: connected
DIRECTION: server -> client
DATA: { clientId: string, server: string, version: string, availableEvents: array }
USE: Sent on connection establishment

EVENT: rebuild_started
DIRECTION: server -> client
DATA: { rebuildId: string, sourcePath: string, totalPhases: number }
USE: Broadcast when rebuild operation begins

EVENT: phase_started
DIRECTION: server -> client
DATA: { rebuildId: string, phase: number, phaseName: string, totalPhases: number }
USE: Broadcast when a phase begins

EVENT: phase_completed
DIRECTION: server -> client
DATA: { rebuildId: string, phase: number, phaseName: string, success: boolean, durationMs: number, details?: object }
USE: Broadcast when a phase finishes

EVENT: rebuild_complete
DIRECTION: server -> client
DATA: { rebuildId: string, status: "success"|"failed"|"rolled_back", documentsTotal: number, documentsSucceeded: number, documentsFailed: number, totalDurationMs: number, error?: string }
USE: Broadcast when rebuild operation finishes

EVENT: rollback_started
DIRECTION: server -> client
DATA: { rebuildId: string, backupId: string, reason: string }
USE: Broadcast when rollback begins

EVENT: rollback_complete
DIRECTION: server -> client
DATA: { rebuildId: string, backupId: string, success: boolean, entriesRestored: number, error?: string }
USE: Broadcast when rollback finishes

CLIENT MESSAGE: subscribe
DATA: { type: "subscribe", events: string[] }
USE: Subscribe to specific events

CLIENT MESSAGE: ping
DATA: { type: "ping" }
RESPONSE: { type: "pong", data: { timestamp: number } }
USE: Keepalive check

---

TOOLS (4)

TOOL: pk_rebuild
INPUT: { source_path: string (required), options?: { dry_run: boolean (default: false), skip_backup: boolean (default: false), force: boolean (default: false), verificationLevel: "basic"|"standard"|"paranoid" (default: "standard") } }
OUTPUT: { rebuildId: string, status: string, phases: array, documentsTotal: number, documentsProcessed: number, documentsSucceeded: number, documentsFailed: number, verificationResults: array, backupPath: string, duration: number }
USE: Orchestrate complete PK rebuild with 7-phase workflow, safety checks, and automatic rollback
EXAMPLE: pk_rebuild({ source_path: "/Users/macbook/Documents/inbox", options: { dry_run: true } })
NOTES: ALWAYS dry_run first. "paranoid" verification runs all 6 tests twice. force=true skips pre-flight warnings.

TOOL: get_rebuild_status
INPUT: { rebuildId: string (required) }
OUTPUT: { success: boolean, rebuild: { id: string, status: string, phases: array, documentsTotal: number, documentsSucceeded: number } }
USE: Check status of a specific rebuild operation
EXAMPLE: get_rebuild_status({ rebuildId: "rebuild_abc123" })
NOTES: Poll during long rebuilds. Status: pending, in_progress, success, failed, rolled_back.

TOOL: list_rebuilds
INPUT: { limit?: number (default: 10), status?: "pending"|"in_progress"|"success"|"failed"|"rolled_back" }
OUTPUT: { success: boolean, count: number, rebuilds: [{ id: string, status: string, startedAt: timestamp, completedAt: timestamp, documentsTotal: number, documentsSucceeded: number }] }
USE: List recent rebuild operations with optional status filter
EXAMPLE: list_rebuilds({ limit: 5, status: "success" })
NOTES: Use to find rebuildId for rollback or status check.

TOOL: rollback_rebuild
INPUT: { rebuildId: string (required), backupId?: string }
OUTPUT: { success: boolean, message: string, backupUsed: string }
USE: Rollback to a previous backup state (uses latest backup if backupId not specified)
EXAMPLE: rollback_rebuild({ rebuildId: "rebuild_abc123" })
NOTES: Uses latest backup if backupId not specified. Rollback is itself a tracked operation.

---

7-PHASE WORKFLOW

1. Pre-flight Safety Checks (source path, disk space, no concurrent rebuild, backup writable)
2. PK Evacuation & Backup (with verification)
3. Document Discovery (scan source path for .md, .txt, .json)
4. Classification Pipeline (GLEC classification via Smart File Organizer)
5. Knowledge Curation (compress via Knowledge Curator, 70-90% reduction)
6. PK Upload (to Notion via PK API)
7. Verification & Rollback (6-test suite, automatic rollback on critical failure)

---

VERIFICATION TESTS

- entry_count: Verify document count matches
- checksum: SHA-256 content verification
- sample_verification: Random sample content check
- glec_distribution: GLEC category balance
- cross_references: Link integrity
- search_index: Index consistency

---

SAFETY GUARANTEES

- 0% data loss guarantee
- Automatic rollback on verification failure
- Multiple backup points
- Concurrent rebuild prevention

---

KEY FILES

SOURCE: /repo/pk-manager/
INDEX: src/index.ts
TYPES: src/types.ts
DATABASE: src/database/schema.ts
BACKUP: src/backup/evacuator.ts
HTTP: src/http/server.ts
WEBSOCKET: src/websocket/server.ts
UTILS: src/utils/checksum.ts, src/utils/logger.ts
TESTS: tests/database.test.ts, tests/rebuild.test.ts, tests/backup.test.ts
CONFIG: config/interlock.json

---

DEPENDENCIES

RUNTIME: @modelcontextprotocol/sdk, better-sqlite3, express, ws, cors, winston, uuid, zod, async-sema
DEV: typescript, vitest, @types/node, @types/better-sqlite3, @types/express, @types/ws, @types/cors, @types/uuid
