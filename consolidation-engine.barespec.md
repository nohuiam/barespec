SERVER: consolidation-engine
VERSION: 1.0.0
UPDATED: 2026-01-02
STATUS: Tested (217 tests, 89% coverage)
PORT: 3032 (UDP/InterLock), 8032 (HTTP), 9032 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk
PURPOSE: Document consolidation and merge operations using BBB redundancy analysis
CONFIG: /repo/consolidation-engine/config/interlock.json
REPO: https://github.com/nohuiam/consolidation-engine
TESTS: 217 (MCP 65, Core 51, Database 20, HTTP 17, WebSocket 9, InterLock 36, Integration 6)

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3) with 3 tables
WORKFLOW: BBB Report → Create Plan → Validate → Detect Conflicts → Merge → Resolve
STRATEGIES: aggressive, conservative, interactive
INTEGRATION: Uses BBB (3008) redundancy reports as input

---

TOOLS (6)

TOOL: create_merge_plan
INPUT: { bbb_report_path: string (required), strategy?: "aggressive"|"conservative"|"interactive" }
OUTPUT: { plan_id: string, clusters: [], estimated_savings: number, file_count: number }
USE: Create merge plan from BBB redundancy analysis report
EXAMPLE: create_merge_plan({ bbb_report_path: "/path/to/bbb-report.json", strategy: "conservative" })
NOTES: Conservative keeps more originals, aggressive merges aggressively

TOOL: validate_plan
INPUT: { plan_id: string (required) }
OUTPUT: { valid: boolean, issues: [], warnings: [] }
USE: Validate merge plan before execution
EXAMPLE: validate_plan({ plan_id: "plan-abc123" })
NOTES: Checks file existence, permissions, conflict potential

TOOL: merge_documents
INPUT: { file_paths: string[] (required), strategy?: "combine"|"prioritize_first"|"prioritize_latest", output_path?: string }
OUTPUT: { merged_file_path: string, content_hash: string, sources_merged: number }
USE: Execute document merge with specified strategy
EXAMPLE: merge_documents({ file_paths: ["/a.md", "/b.md"], strategy: "combine" })
NOTES: Auto-generates output path if not specified

TOOL: detect_conflicts
INPUT: { file_paths: string[] (required) }
OUTPUT: { conflicts: [], conflict_count: number, can_auto_resolve: boolean }
USE: Detect conflicts between files before merging
EXAMPLE: detect_conflicts({ file_paths: ["/doc1.md", "/doc2.md", "/doc3.md"] })
NOTES: Detects content, structural, and metadata conflicts

TOOL: resolve_conflicts
INPUT: { conflict_id: string (required), resolution: "keep_first"|"keep_latest"|"merge_both"|"custom", custom_content?: string }
OUTPUT: { success: boolean, resolution_applied: string, result_path: string }
USE: Resolve detected conflicts with specified strategy
EXAMPLE: resolve_conflicts({ conflict_id: "conflict-xyz", resolution: "merge_both" })

TOOL: get_merge_history
INPUT: { limit?: number, since?: string, plan_id?: string }
OUTPUT: { operations: [], total: number }
USE: Get merge operation history
EXAMPLE: get_merge_history({ limit: 20 })

---

LAYERS

1. MCP stdio (6 tools) - Primary interface
2. InterLock UDP mesh (port 3032) - Peer communication
3. HTTP REST API (port 8032) - External integrations
4. WebSocket real-time (port 9032) - Live updates

---

HTTP REST API

GET  /health                    → { status, plans_count, active_merges }
POST /api/plans                 → Create merge plan
GET  /api/plans/:id             → Get plan details
POST /api/plans/:id/validate    → Validate plan
POST /api/merge                 → Execute merge
GET  /api/conflicts             → List conflicts
POST /api/conflicts/:id/resolve → Resolve conflict
GET  /api/history               → Get merge history

---

WEBSOCKET EVENTS

S→C plan_created               → New plan created
S→C merge_started              → Merge operation started
S→C merge_progress             → Merge progress update
S→C merge_complete             → Merge completed
S→C conflict_detected          → Conflict found
S→C conflict_resolved          → Conflict resolved
S→C error                      → Error occurred
C↔S ping/pong                  → Keepalive

---

INTERLOCK SIGNALS

Emits: HEARTBEAT, MERGE_STARTED, MERGE_COMPLETE, CONFLICT_DETECTED, PLAN_CREATED
Accepts: HEARTBEAT, DISCOVERY, SHUTDOWN, HEALTH_CHECK, MERGE_REQUEST

PEERS: safe-batch-processor (3022), project-context (3016), snapshot (3003), smart-file-organizer (3007), bonzai-bloat-buster (3008)

---

DATABASE SCHEMA

TABLE: merge_plans
COLUMNS: id, plan_id, bbb_report_path, strategy, status, clusters_json, estimated_savings, created_at, executed_at
INDEXES: idx_plans_id, idx_plans_status

TABLE: merge_operations
COLUMNS: id, plan_id, operation_type, source_paths_json, output_path, content_hash, status, created_at, completed_at
INDEXES: idx_operations_plan, idx_operations_status

TABLE: merge_conflicts
COLUMNS: id, plan_id, conflict_type, file_paths_json, description, resolution, resolved_at, resolved_by
INDEXES: idx_conflicts_plan, idx_conflicts_type

---

KEY FILES

SOURCE: /repo/consolidation-engine/
INDEX: src/index.ts
CORE: src/core/ (plan-manager.ts, merge-engine.ts, conflict-resolver.ts)
TOOLS: src/tools/
INTERLOCK: src/interlock/socket.ts
HTTP: src/http/server.ts
WEBSOCKET: src/websocket/server.ts
DATABASE: data/consolidation-engine.db
CONFIG: config/interlock.json

DEPENDENCIES: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk

---

SERVICES

- PlanManager: Plan creation & validation from BBB analysis
- MergeEngine: Document merging with strategy selection
- ConflictResolver: Conflict detection and resolution handling

---

MERGE STRATEGIES

| Strategy | Behavior |
|----------|----------|
| combine | Merge all content sections |
| prioritize_first | Keep first file's structure, append others |
| prioritize_latest | Keep newest file's structure |

---

COVERAGE

| Metric | Coverage |
|--------|----------|
| Statements | 89.36% |
| Branches | 76.30% |
| Functions | 90.90% |
| Lines | 89.34% |

---

CI/CD

- GitHub Actions CI pipeline
- Tested on Node 18.x, 20.x, 22.x
- All 217 tests passing

---

NOTES

1. Uses BBB redundancy reports as input for intelligent merging
2. Three merge strategies: aggressive, conservative, interactive
3. Full conflict detection before merge execution
4. Automatic backup via snapshot (3003) integration
5. 89% code coverage with 217 automated tests
6. CI/CD pipeline on GitHub Actions
