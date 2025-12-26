SERVER: pk-manager
VERSION: 1.1
UPDATED: 2025-12-26
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

TOOLS (4)

TOOL: pk_rebuild
INPUT: { source_path: string (required), options?: { dry_run: boolean (default: false), skip_backup: boolean (default: false), force: boolean (default: false), verificationLevel: "basic"|"standard"|"paranoid" (default: "standard") } }
OUTPUT: { rebuildId, status, phases[], documentsTotal, documentsProcessed, documentsSucceeded, documentsFailed, verificationResults[], backupPath, duration }
USE: Orchestrate complete PK rebuild with 7-phase workflow, safety checks, and automatic rollback
EXAMPLE: pk_rebuild({ source_path: "/Users/macbook/Documents/inbox", options: { dry_run: true } })

TOOL: get_rebuild_status
INPUT: { rebuildId: string (required) }
OUTPUT: { success: boolean, rebuild: { id, status, phases[], documentsTotal, documentsSucceeded } }
USE: Check status of a specific rebuild operation
EXAMPLE: get_rebuild_status({ rebuildId: "rebuild_abc123" })

TOOL: list_rebuilds
INPUT: { limit?: number (default: 10), status?: "pending"|"in_progress"|"success"|"failed"|"rolled_back" }
OUTPUT: { success: boolean, count, rebuilds: [{ id, status, startedAt, completedAt, documentsTotal, documentsSucceeded }] }
USE: List recent rebuild operations with optional status filter
EXAMPLE: list_rebuilds({ limit: 5, status: "success" })

TOOL: rollback_rebuild
INPUT: { rebuildId: string (required), backupId?: string }
OUTPUT: { success: boolean, message, backupUsed }
USE: Rollback to a previous backup state (uses latest backup if backupId not specified)
EXAMPLE: rollback_rebuild({ rebuildId: "rebuild_abc123" })

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
UTILS: src/utils/checksum.ts, src/utils/logger.ts

DEPENDENCIES: project-context, knowledge-curator, PK API (Notion)
