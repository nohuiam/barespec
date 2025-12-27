SERVER: snapshot
VERSION: 1.1
UPDATED: 2025-12-27
STATUS: Production
PORT: 3003 (UDP/InterLock), 8003 (HTTP), 9003 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: System state capture and integrity verification for disaster recovery
CONFIG: /repo/snapSHOT/config/interlock.json

---

ARCHITECTURE

DATABASE: JSON file-based (SQLite planned)
LAYERS: MCP (stdio), HTTP (REST), WebSocket, InterLock (UDP mesh)
NICKNAME: Memory Defense AstroSentry

---

TOOLS (6)

TOOL: snapshot_capture_snapshot
INPUT: { type?: "full"|"incremental"|"config" (default: "full"), label?: string }
OUTPUT: { snapshot_id: number, type: string, label: string, timestamp: string, file_count: number, total_size_bytes: number, checksums: object }
USE: Manually trigger snapshot capture of current system state
EXAMPLE: snapshot_capture_snapshot({ type: "full", label: "pre-migration-backup" })
NOTES: "full" captures everything. "incremental" only changes since last. "config" only configuration files.

TOOL: snapshot_verify_integrity
INPUT: { snapshot_id?: number }
OUTPUT: { verified: boolean, snapshot_id: number, mismatches: array, corrupted_files: array, missing_files: array, timestamp: string }
USE: Verify current system state against snapshot to detect corruption/drift
EXAMPLE: snapshot_verify_integrity({ snapshot_id: 42 })
NOTES: Uses latest snapshot if id not specified. Lists all discrepancies.

TOOL: snapshot_list_snapshots
INPUT: { limit?: number (default: 50), type?: "full"|"incremental"|"config", status?: "captured"|"verified"|"corrupted" }
OUTPUT: { snapshots: [{ id: number, type: string, label: string, status: string, timestamp: string, file_count: number }], total: number }
USE: List all captured snapshots with optional filtering
EXAMPLE: snapshot_list_snapshots({ limit: 10, type: "full" })

TOOL: snapshot_get_snapshot_details
INPUT: { snapshot_id: number (required), include_files?: boolean (default: false), include_verifications?: boolean (default: true) }
OUTPUT: { id: number, type: string, label: string, status: string, timestamp: string, file_count: number, total_size: number, files?: array, verifications: array }
USE: Get detailed information about a specific snapshot
EXAMPLE: snapshot_get_snapshot_details({ snapshot_id: 42, include_files: false })
NOTES: include_files can return very large response.

TOOL: snapshot_compare_snapshots
INPUT: { snapshot_id_1: number (required), snapshot_id_2: number (required) }
OUTPUT: { comparison: { added: array, removed: array, modified: array, unchanged_count: number }, summary: object }
USE: Compare two snapshots to identify changes between them
EXAMPLE: snapshot_compare_snapshots({ snapshot_id_1: 40, snapshot_id_2: 42 })
NOTES: snapshot_id_1 should be earlier than snapshot_id_2.

TOOL: snapshot_restore_snapshot
INPUT: { snapshot_id: number (required), confirm: boolean (required), create_backup?: boolean (default: true) }
OUTPUT: { success: boolean, snapshot_id: number, files_restored: number, backup_created: boolean, backup_id: number, timestamp: string }
USE: Restore system state from snapshot (DESTRUCTIVE - requires confirm: true)
EXAMPLE: snapshot_restore_snapshot({ snapshot_id: 42, confirm: true, create_backup: true })
NOTES: REQUIRES confirm: true. Creates backup before restore by default.

---

SNAPSHOT TYPES

- full: Complete system state
- incremental: Changes since last full snapshot
- config: Configuration files only

STATUS: captured, verified, corrupted

---

VERIFICATION

- SHA-256 checksums
- File presence verification
- Size matching
- Corruption detection

---

SAFETY

- Requires confirmation for restore
- Auto-backup before restore
- Graceful shutdown handlers

---

KEY FILES

SOURCE: /repo/snapSHOT/
INDEX: src/index.js
CORE: src/core/SnapshotManager.js
TOOLS: src/tools/index.js
DATABASE: src/database/index.js
MCP: src/mcp/server.js
HTTP: src/http/server.js
WEBSOCKET: src/websocket/server.js
MESH: src/mesh/socket.js, src/mesh/handlers.js, src/mesh/protocol.js
CONFIG: config/server.json, config/interlock.json
