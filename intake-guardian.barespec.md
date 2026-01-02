SERVER: intake-guardian
VERSION: 1.0.0
UPDATED: 2026-01-02
STATUS: Tested (27 tests passed)
PORT: 3023 (UDP/InterLock), 8023 (HTTP), 9023 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk
PURPOSE: Gate-keeper for content admission using BBB redundancy scores and GNC-004 thresholds
CONFIG: /repo/intake-guardian/config/interlock.json
TESTS: 27 (check_content, check_file, admit_content, history, config, errors, websocket)

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3)
WORKFLOW: Content → BBB redundancy check → GNC-004 threshold evaluation → admission decision
INTEGRATION: Depends on bonzai-bloat-buster (8008) for redundancy scores
FALLBACK: If BBB unavailable, score defaults to 0 (auto-admit)

---

TOOLS (5)

TOOL: check_content
INPUT: { content: string (required), content_type?: string, metadata?: object }
OUTPUT: { decision: string, redundancy_score: number, similar_items: [], reason: string, can_override: boolean, content_hash: string }
USE: Check raw content for admission eligibility
EXAMPLE: check_content({ content: "Some text to evaluate" })
NOTES: Returns content_hash needed for admit_content

TOOL: check_file
INPUT: { file_path: string (required) }
OUTPUT: { decision: string, redundancy_score: number, similar_files: [], file_info: object, reason: string, can_override: boolean, content_hash: string }
USE: Check file for admission eligibility
EXAMPLE: check_file({ file_path: "/path/to/file.md" })
NOTES: Reads file content and evaluates against existing corpus

TOOL: admit_content
INPUT: { content_hash: string (required), override?: boolean, override_reason?: string, destination?: string }
OUTPUT: { admitted: boolean, admission_id: string, timestamp: string, destination: string }
USE: Admit content after check, or override rejection
EXAMPLE: admit_content({ content_hash: "abc123", override: true, override_reason: "Manual review passed" })
NOTES: Requires content_hash from prior check_content/check_file call

TOOL: get_intake_history
INPUT: { limit?: number, decision_filter?: string, since?: string }
OUTPUT: { entries: [], total: number }
USE: Get audit history of admission decisions
EXAMPLE: get_intake_history({ limit: 50, decision_filter: "auto_reject" })

TOOL: configure_thresholds
INPUT: { auto_admit_max?: number, review_recommended_max?: number, review_required_max?: number }
OUTPUT: { thresholds: object, updated: boolean }
USE: Set admission thresholds (GNC-004 boundaries)
EXAMPLE: configure_thresholds({ auto_admit_max: 30, review_required_max: 85 })

---

GNC-004 THRESHOLDS (Default)

| Score Range | Decision |
|-------------|----------|
| 0-30% | auto_admit |
| 31-70% | review_recommended |
| 71-85% | review_required |
| 86-100% | auto_reject |

---

LAYERS

1. MCP stdio (5 tools) - Primary interface
2. InterLock UDP mesh (port 3023) - Peer communication
3. HTTP REST API (port 8023) - External integrations
4. WebSocket real-time (port 9023) - Live updates

---

HTTP REST API

GET  /health                → { status, bbb_connected, db_stats }
GET  /stats                 → { total_checked, admitted, rejected, pending }
POST /api/check             → { decision, redundancy_score, ... }
POST /api/admit             → { admitted, admission_id }
GET  /api/history           → { entries[], total }
GET  /api/config            → { thresholds }
PUT  /api/config            → { thresholds, updated }

---

WEBSOCKET EVENTS

S→C content_checked         → Emitted when content is checked
S→C content_admitted        → Emitted when content is admitted
S→C content_rejected        → Emitted when content is rejected
S→C threshold_updated       → Emitted when thresholds change
S→C error                   → Emitted on errors
C↔S ping/pong               → Keepalive

---

INTERLOCK SIGNALS

Emits: HEARTBEAT (0x01), CONTENT_CHECKED (0x10), CONTENT_ADMITTED (0x20), CONTENT_REJECTED (0x21)
Accepts: HEARTBEAT, DISCOVERY, SHUTDOWN, HEALTH_CHECK, HEALTH_RESPONSE

---

DATABASE SCHEMA

TABLE: intake_decisions
COLUMNS: id, content_hash, decision, redundancy_score, reason, checked_at, metadata
INDEXES: idx_decisions_hash, idx_decisions_date

TABLE: admitted_content
COLUMNS: id, admission_id, content_hash, admitted_at, destination, override, override_reason
INDEXES: idx_admitted_hash, idx_admitted_date

TABLE: config
COLUMNS: key, value, updated_at
INDEXES: PRIMARY KEY (key)

---

KEY FILES

SOURCE: /repo/intake-guardian/
INDEX: src/index.ts
SERVICES: src/services/
INTERLOCK: src/interlock/socket.ts
HTTP: src/http/server.ts
WEBSOCKET: src/websocket/server.ts
DATABASE: data/intake-guardian.db
CONFIG: config/interlock.json

DEPENDENCIES: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk

---

INTEGRATIONS

BBB (bonzai-bloat-buster): HTTP call to localhost:8008 for redundancy analysis
FALLBACK: If BBB unavailable, defaults to score=0 (auto-admit)

---

NOTES

1. Content must pass check before admit - admits require content_hash from prior check
2. Override capability allows manual review to bypass auto-reject
3. All decisions logged for audit trail
4. GNC-004 thresholds are configurable per-instance
