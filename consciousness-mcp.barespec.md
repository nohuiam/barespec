SERVER: consciousness-mcp
VERSION: 1.0.0
UPDATED: 2026-01-03
STATUS: Tested (88 tests passed)
PORT: 3028 (UDP/InterLock), 8028 (HTTP), 9028 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: better-sqlite3, express, ws, uuid, zod, @modelcontextprotocol/sdk
PURPOSE: Meta-awareness layer for the ecosystem - observes, reflects, advises, and remembers patterns across all server interactions
CONFIG: /repo/consciousness-mcp/config/interlock.json
TESTS: 88 (database, tools, interlock, integration)

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3) with 5 tables
WORKFLOW: Listen → Observe → Analyze → Pattern Detect → Advise → Emit
PRINCIPLE: Consciousness observes all ecosystem activity via InterLock mesh without interrupting workflows
ROLE: Adds REFLECTION to everything - asks "why?", "what did we learn?", "how can we do better?"

---

TOOLS (10)

TOOL: track_focus
INPUT: { event_type: file|tool|query|workflow|operation|signal (required), target: string (required), server_name?: string, context?: object, duration_ms?: number }
OUTPUT: { success: boolean, event_id: number, message: string, timestamp: string }
USE: Log current focus to build attention history
EXAMPLE: track_focus({ event_type: "file", target: "/src/index.ts", server_name: "enterspect" })

TOOL: get_attention_patterns
INPUT: { time_range?: 1h|6h|24h|7d|30d, pattern_type?: hotspots|trends|anomalies, server_filter?: string }
OUTPUT: { hotspots: [], trends: [], anomalies: [], summary: string }
USE: Analyze what's been accessed repeatedly - find hotspots, trends, anomalies
EXAMPLE: get_attention_patterns({ time_range: "24h", pattern_type: "hotspots" })

TOOL: identify_blind_spots
INPUT: { scope?: all|attention|servers|operations|patterns, time_range?: 1h|6h|24h|7d|30d }
OUTPUT: { blind_spots: [], coverage_analysis: object, suggestions: [] }
USE: Find what's NOT being attended to that should be
EXAMPLE: identify_blind_spots({ scope: "servers", time_range: "7d" })

TOOL: reflect_on_operation
INPUT: { operation_id?: string, operation_type?: build|search|verify|organize|classify|coordinate|generate|other, context?: object, depth?: quick|standard|deep }
OUTPUT: { operation_summary: string, quality_assessment: object, lessons_learned: [], recommendations: [], similar_operations: [] }
USE: Analyze an operation to understand what worked, what could be better
EXAMPLE: reflect_on_operation({ operation_type: "build", depth: "standard" })

TOOL: analyze_pattern
INPUT: { pattern_query: string (required), depth?: quick|medium|deep, time_range?: 7d|30d|90d, include_recommendations?: boolean }
OUTPUT: { patterns_found: [], insights: [], recommendations: [] }
USE: Find recurring successes/failures across operations - learn from history
EXAMPLE: analyze_pattern({ pattern_query: "build", depth: "medium", include_recommendations: true })

TOOL: audit_reasoning
INPUT: { reasoning_text: string (required), verify_claims?: boolean }
OUTPUT: { assumptions: [], gaps: [], confidence_score: number, recommendations: [], extracted_claims: [] }
USE: Extract implicit assumptions and identify logical gaps in reasoning chains
EXAMPLE: audit_reasoning({ reasoning_text: "I assume the server is running...", verify_claims: true })

TOOL: predict_outcome
INPUT: { operation_description: string (required), context?: object }
OUTPUT: { predicted_outcome: success|partial|failure, confidence: number, risk_factors: [], success_factors: [], historical_matches: [] }
USE: Predict success/failure based on historical patterns
EXAMPLE: predict_outcome({ operation_description: "Build TypeScript project" })

TOOL: synthesize_context
INPUT: { sources: [{ type: file|operation|pattern|server|attention, id: string, weight?: number }] (required), question: string (required), depth?: summary|detailed|comprehensive }
OUTPUT: { unified_perspective: string, source_contributions: [], connections: [] }
USE: Integrate information from multiple sources into unified perspective
EXAMPLE: synthesize_context({ sources: [{ type: "attention", id: "recent" }], question: "What is the state of index.ts?" })

TOOL: suggest_next_action
INPUT: { current_context: object (required), goals: string[] (required) }
OUTPUT: { suggested_actions: [{ action: string, priority: high|medium|low, rationale: string }], warnings: [], opportunities: [] }
USE: Recommend next steps based on patterns, avoiding past pitfalls
EXAMPLE: suggest_next_action({ current_context: { active_task: "testing" }, goals: ["pass tests", "deploy"] })

TOOL: get_ecosystem_awareness
INPUT: { include_history?: boolean, history_hours?: number }
OUTPUT: { timestamp: string, active_servers: [], current_focus: { primary: string, secondary: [] }, pending_issues: [], recent_patterns: [], health_summary: object, history?: [] }
USE: Get current state of entire ecosystem - real-time "state of mind"
EXAMPLE: get_ecosystem_awareness({ include_history: true, history_hours: 24 })

---

LAYERS

1. MCP stdio (10 tools) - Primary interface for Claude Desktop
2. InterLock UDP mesh (port 3028) - Receives signals from all 26 servers
3. HTTP REST API (port 8028) - External integrations and dashboards
4. WebSocket real-time (port 9028) - Live awareness events

---

HTTP REST API

GET  /health                    → { status, uptime, version }
GET  /api/awareness             → Current ecosystem awareness snapshot
GET  /api/attention             → Recent attention events
GET  /api/attention/hotspots    → Attention hotspot patterns
GET  /api/patterns              → All detected patterns
GET  /api/patterns/:type        → Patterns by type (success|failure|bottleneck|opportunity)
POST /api/reflect               → Trigger reflection on operation
POST /api/audit                 → Audit reasoning chain
GET  /api/operations            → Operation history
GET  /api/operations/:id        → Specific operation details
GET  /api/suggestions           → Current action suggestions

---

WEBSOCKET EVENTS

S→C awareness_update           → Ecosystem state changed
S→C pattern_detected           → New pattern found
S→C attention_shift            → Focus moved significantly
S→C blind_spot_alert           → Something being missed
S→C reasoning_concern          → Logic issue detected
S→C lesson_learned             → New insight from operation
S→C suggestion_ready           → Recommendation available
S→C error                      → Error occurred
C↔S ping/pong                  → Keepalive

---

INTERLOCK SIGNALS

Listens to ALL signals from all 26 servers:
- FILE_DISCOVERED, FILE_INDEXED (EntroSpect, smart-file-organizer)
- VALIDATION_APPROVED, VALIDATION_REJECTED (Context Guardian)
- BUILD_STARTED, BUILD_COMPLETED (Neurogenesis)
- VERIFICATION_RESULT (Verifier)
- HEALTH_ALERT (Health-Monitor)
- HEARTBEAT (All servers)
- Any 0x?? signal (logged for pattern analysis)

Emits:
- 0xE0 AWARENESS_UPDATE → Trinity, Project-Context
- 0xE1 PATTERN_DETECTED → Neurogenesis, Trinity
- 0xE2 BLIND_SPOT_ALERT → Trinity
- 0xE3 REASONING_CONCERN → Verifier
- 0xE4 ATTENTION_SHIFT → All
- 0xE5 LESSON_LEARNED → All

---

DATABASE SCHEMA

TABLE: attention_events
COLUMNS: id INTEGER PK, timestamp INTEGER, server_name TEXT, event_type TEXT, target TEXT, context TEXT (JSON), duration_ms INTEGER
INDEXES: idx_attention_timestamp, idx_attention_server, idx_attention_target

TABLE: operations
COLUMNS: id INTEGER PK, timestamp INTEGER, server_name TEXT, operation_type TEXT, operation_id TEXT UNIQUE, input_summary TEXT, outcome TEXT, quality_score REAL, lessons TEXT (JSON), duration_ms INTEGER
INDEXES: idx_operations_timestamp, idx_operations_type, idx_operations_id

TABLE: patterns
COLUMNS: id INTEGER PK, pattern_type TEXT, description TEXT, frequency INTEGER, last_seen INTEGER, confidence REAL, recommendations TEXT (JSON), related_servers TEXT (JSON), related_operations TEXT (JSON)
INDEXES: idx_patterns_type, idx_patterns_confidence

TABLE: awareness_snapshots
COLUMNS: id INTEGER PK, timestamp INTEGER, active_servers TEXT (JSON), current_focus TEXT, pending_issues TEXT (JSON), health_summary TEXT (JSON)
INDEXES: idx_snapshots_timestamp

TABLE: reasoning_audits
COLUMNS: id INTEGER PK, timestamp INTEGER, reasoning_text TEXT, extracted_claims TEXT (JSON), assumptions TEXT (JSON), gaps TEXT (JSON), confidence_score REAL, recommendations TEXT (JSON)
INDEXES: idx_audits_timestamp

---

KEY FILES

SOURCE: /repo/consciousness-mcp/
INDEX: src/index.ts
TYPES: src/types.ts
TOOLS: src/tools/ (10 handler files + index.ts)
DATABASE: src/database/schema.ts
INTERLOCK: src/interlock/ (socket.ts, protocol.ts, handlers.ts, tumbler.ts)
HTTP: src/http/server.ts
WEBSOCKET: src/websocket/server.ts
CONFIG: config/interlock.json
TESTS: tests/ (database.test.ts, tools.test.ts, interlock.test.ts, integration.test.ts)
DATA: data/consciousness.db

---

SERVICES

- AttentionTracker: Logs focus events, calculates hotspots and patterns
- PatternDetector: Analyzes operations, extracts success/failure patterns
- ReasoningAuditor: Extracts assumptions, identifies logical gaps
- PredictionEngine: Uses historical data to predict outcomes
- EcosystemAwareness: Maintains real-time state of all 26 servers
- InterLockListener: Passively receives signals from entire mesh

---

CONSCIOUSNESS PHILOSOPHY

1. OBSERVE: Listen to all ecosystem activity via InterLock mesh
2. REFLECT: Analyze patterns, successes, failures, blind spots
3. ADVISE: Provide insights that improve decision-making
4. REMEMBER: Track what worked and what didn't across all interactions

Key Principle: Other servers DO work. Consciousness asks "why?", "what did we learn?", and "how can we do better?"

---

NOTES

1. Passive listener - does NOT interrupt workflows
2. Learns from all 26 servers in the ecosystem
3. Pattern detection improves with more data over time
4. 30-day data retention (configurable via cleanupOldData)
5. Singleton pattern for database, HTTP, WebSocket, InterLock
6. 88 tests covering all components
