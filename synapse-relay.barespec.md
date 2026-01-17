SERVER: synapse-relay
VERSION: 1.0.0
UPDATED: 2026-01-08
STATUS: Tested (10 HTTP tests passed)
PORT: 3025 (UDP/InterLock), 8025 (HTTP), 9025 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk, cors, express-rate-limit
PURPOSE: InterLock mesh signal routing - the "switchboard operator" for neural packet routing
CONFIG: /repo/synapse-relay/config/interlock.json
TESTS: 10 HTTP tests (health, 404, CORS x6, rate limit x2)

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3) with 4 tables
WORKFLOW: Signal → Rule Match → Route/Buffer → Track → Aggregate Stats
BUFFERING: Signals to offline targets buffered with TTL and retry
PEERS: 17 mesh peers configured
RETRY: Exponential backoff (1s, 5s, 15s), max 3 retries

---

TOOLS (4)

TOOL: relay_signal
INPUT: { signal_type: string (required), payload: object (required), targets: string[] (required), priority?: number }
OUTPUT: { relay_id: string, delivered: [], buffered: [], failed: [] }
USE: Relay a signal to one or more target servers
EXAMPLE: relay_signal({ signal_type: "STATUS_UPDATE", payload: { status: "healthy" }, targets: ["context-guardian", "trinity-coordinator"] })
NOTES: Multicast supported, offline targets auto-buffered

TOOL: configure_relay
INPUT: { action: "add"|"update"|"remove"|"list" (required), rule?: object }
OUTPUT: { rules: [], affected?: number }
USE: Configure automatic relay rules for signal routing
EXAMPLE: configure_relay({ action: "add", rule: { pattern: "HEALTH_*", targets: ["health-monitor"], priority: 10 } })
NOTES: Rules match by signal pattern, routed automatically

TOOL: get_relay_stats
INPUT: { grouping?: "server"|"signal"|"hour", since?: string }
OUTPUT: { stats: [], totals: object }
USE: Get relay statistics and performance metrics
EXAMPLE: get_relay_stats({ grouping: "server", since: "2026-01-01" })

TOOL: buffer_signals
INPUT: { action: "list"|"retry"|"clear"|"flush" (required), target?: string }
OUTPUT: { buffered: [], affected?: number }
USE: Manage buffered signals for offline targets
EXAMPLE: buffer_signals({ action: "retry", target: "snapshot" })
NOTES: Flush delivers all buffered signals immediately

---

LAYERS

1. MCP stdio (4 tools) - Primary interface
2. InterLock UDP mesh (port 3025) - Peer communication
3. HTTP REST API (port 8025) - External integrations
4. WebSocket real-time (port 9025) - Live updates

---

HTTP REST API

GET  /health                → { status, active_relays, buffer_count }
POST /api/relay             → Relay single signal
POST /api/relay/multicast   → Relay to multiple targets
GET  /api/relay/:id         → Get relay details
POST /api/rules             → Create relay rule
GET  /api/rules             → List relay rules
PUT  /api/rules/:id         → Update relay rule
DELETE /api/rules/:id       → Delete relay rule
GET  /api/stats             → Get relay statistics
GET  /api/buffer            → List buffered signals
POST /api/buffer/retry      → Retry buffered signals
POST /api/buffer/flush      → Flush buffer to targets

---

WEBSOCKET EVENTS

S→C relay:sent              → Signal relay succeeded
S→C relay:failed            → Signal relay failed
S→C relay:buffered          → Signal buffered (target offline)
S→C buffer:retry            → Buffer retry attempt
S→C buffer:expired          → Buffered signal expired (TTL)
S→C stats:update            → Hourly stats aggregation
C↔S ping/pong               → Keepalive

---

INTERLOCK SIGNALS

Incoming: PING, HEARTBEAT, RELAY_REQUEST, RELAY_RESPONSE
Outgoing: PONG, RELAY_REQUEST, RELAY_RESPONSE, RELAY_FAILED

---

DATABASE SCHEMA

TABLE: signal_relays
COLUMNS: id, relay_id, signal_type, source, targets_json, payload_json, status, created_at, completed_at
INDEXES: idx_relays_id, idx_relays_signal, idx_relays_status

TABLE: relay_rules
COLUMNS: id, pattern, targets_json, priority, transform_json, enabled, created_at, updated_at
INDEXES: idx_rules_pattern, idx_rules_priority

TABLE: signal_buffer
COLUMNS: id, relay_id, target, signal_type, payload_json, retry_count, last_retry, expires_at, created_at
INDEXES: idx_buffer_target, idx_buffer_expires

TABLE: relay_stats
COLUMNS: id, period_start, server_name, signal_type, relayed_count, failed_count, avg_latency_ms
INDEXES: idx_stats_period, idx_stats_server

---

KEY FILES

SOURCE: /repo/synapse-relay/
INDEX: src/index.ts
RELAY: src/relay/ (engine.ts, buffer-manager.ts, rule-engine.ts, stats-aggregator.ts)
TOOLS: src/tools/
INTERLOCK: src/interlock/socket.ts
HTTP: src/http/server.ts (12 endpoints)
WEBSOCKET: src/websocket/server.ts (6 events)
DATABASE: data/synapse-relay.db
CONFIG: config/interlock.json

DEPENDENCIES: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk

---

SERVICES

- RelayEngine: Core signal routing logic
- BufferManager: Signal buffering with TTL and retry
- RuleEngine: Pattern matching and auto-routing
- StatsAggregator: Hourly statistics collection

---

TIMING

| Setting | Value |
|---------|-------|
| Buffer TTL | 24 hours |
| Retry intervals | 1s, 5s, 15s |
| Max retries | 3 |
| Stats aggregation | hourly |
| Buffer processor | every 5 seconds |

---

SECURITY

CORS: Origin whitelist (localhost:5173, 127.0.0.1:5173, localhost:3099, localhost:8025)
RATE_LIMITS:
- General: 100 requests/minute
- Relay: 50 requests/minute
HEADERS: RateLimit-Limit, RateLimit-Remaining, RateLimit-Reset (draft-7)
AUDIT: Linus audit completed 2026-01-08 (Instance 8)

---

NOTES

1. Central hub for InterLock mesh signal routing
2. 17 mesh peers configured
3. Automatic buffering for offline targets
4. Rule-based auto-routing with priority
5. Payload transformation support
6. Exponential backoff on retries
