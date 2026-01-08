SERVER: intelligent-router
VERSION: 1.0.0
UPDATED: 2026-01-07
STATUS: Production
PORT: 3020 (UDP/InterLock), 8020 (HTTP), 9020 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: Claude API (intent classification - optional)
PURPOSE: Natural language to automated multi-tool workflow routing
CONFIG: /repo/intelligentrouter/config/interlock.json

---

ARCHITECTURE

WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
FUNCTION: Intent classification and workflow orchestration
LEARNING: Pattern recognition from successful routings
COMPLEXITY: 1-5 scale (1=single tool, 5=complex multi-step)
INTERLOCK: BaNano protocol, signals 0xC0-0xCF (ROUTE_REQUEST, ROUTE_RESPONSE, ROUTE_COMPLETE)

---

TOOLS (2)

TOOL: route_request
INPUT: { request: string (required), context?: { currentPath?: string, recentOperations?: string[], timeConstraints?: string, userPreferences?: object }, routing?: { autoExecute?: boolean, preferredTools?: string[], maxComplexity?: 1-5 }, learning?: { trackPerformance?: boolean, savePattern?: boolean } }
OUTPUT: { intent: string, confidence: number, workflow: { steps: [{ tool: string, params: object, expected_output: string }], estimated_duration: number }, alternativeWorkflows: array, routingDecision: object }
USE: Route natural language request to appropriate tools with optimal workflow
EXAMPLE: route_request({ request: "organize my inbox safely", routing: { autoExecute: false } })
NOTES: autoExecute=true runs workflow if confidence > 0.85. maxComplexity limits workflow depth.

TOOL: get_routing_history
INPUT: { limit?: number (default: 10), offset?: number (default: 0), intent?: string, status?: string }
OUTPUT: { history: [{ request: string, intent: string, workflow: object, execution_time: number, status: string }], total: number, patterns: array }
USE: View past routing decisions and patterns for learning and debugging
EXAMPLE: get_routing_history({ limit: 20, status: "completed" })
NOTES: patterns shows frequently used request->workflow mappings.

---

INTENTS

- file_organization: Organize, sort, move files
- classification: GLEC categorization
- search: Find files, content, patterns
- analysis: Analyze documents, detect issues
- batch_processing: Bulk operations
- maintenance: Cleanup, deduplication, archival

---

COMPLEXITY LEVELS

- 1: Single tool, direct execution
- 2: Two tools, sequential
- 3: Multiple tools, some parallel
- 4: Complex workflow, conditionals
- 5: Multi-phase, requires validation

LEARNING: Tracks successful patterns for future routing optimization

---

HTTP REST API (port 8020)

ENDPOINT: GET /health
OUTPUT: { status: string, server: string, version: string, uptime: object, ports: object, routing: object, timestamp: string }
USE: Health check with routing statistics

ENDPOINT: GET /stats
OUTPUT: { success: boolean, stats: { routing: object, patterns: object, monitoring: object }, timestamp: string }
USE: Comprehensive routing and pattern statistics

ENDPOINT: GET /api/v1/routing/history
OUTPUT: { success: boolean, count: number, history: array, pagination: object, timestamp: string }
USE: Query routing history with pagination and filters

ENDPOINT: GET /api/v1/routing/:requestId
OUTPUT: { success: boolean, routing: object, timestamp: string }
USE: Get specific routing by request ID

ENDPOINT: POST /api/v1/routing/request
INPUT: { request: string, context?: object, routing?: object, learning?: object }
OUTPUT: { success: boolean, result: object, timestamp: string }
USE: Route a request via HTTP

ENDPOINT: POST /api/v1/cleanup
OUTPUT: { success: boolean, message: string, timestamp: string }
USE: Trigger database cleanup

---

WEBSOCKET EVENTS (port 9020)

EVENT: connected
DIRECTION: server -> client
DATA: { clientId: string, server: string, version: string, routing: object, availableEvents: array }
USE: Sent on connection establishment

EVENT: routing_started
DIRECTION: server -> client
DATA: { requestId: string, rawRequest: string, timestamp: string }
USE: Broadcast when routing request begins

EVENT: intent_detected
DIRECTION: server -> client
DATA: { requestId: string, intent: string, confidence: number, timestamp: string }
USE: Broadcast when intent analysis completes

EVENT: pattern_matched
DIRECTION: server -> client
DATA: { requestId: string, patternId: string, similarity: number, timestamp: string }
USE: Broadcast when learned pattern matches

EVENT: tools_selected
DIRECTION: server -> client
DATA: { requestId: string, tools: array, scores: object, timestamp: string }
USE: Broadcast when tools are selected

EVENT: execution_plan_ready
DIRECTION: server -> client
DATA: { requestId: string, plan: object, coordinationPattern: string, timestamp: string }
USE: Broadcast when execution plan is ready

EVENT: routing_complete
DIRECTION: server -> client
DATA: { requestId: string, success: boolean, durationMs: number, timestamp: string }
USE: Broadcast when routing completes

EVENT: error_occurred
DIRECTION: server -> client
DATA: { requestId: string, error: string, recoverable: boolean, timestamp: string }
USE: Broadcast on routing error

EVENT: stats_update
DIRECTION: server -> client
DATA: { routing: object, patterns: object, timestamp: string }
USE: Periodic stats broadcast (30s)

CLIENT MESSAGE: subscribe
DATA: { type: "subscribe", events: string[] }
USE: Subscribe to specific events

CLIENT MESSAGE: ping
DATA: { type: "ping" }
RESPONSE: { type: "pong", data: { timestamp: number } }
USE: Keepalive check

---

KEY FILES

SOURCE: /repo/intelligentrouter/
INDEX: src/index.ts
ROUTER: src/services/intelligentRouter.ts
HTTP: src/http/server.ts
WEBSOCKET: src/websocket/server.ts
TESTS: tests/http.test.ts, tests/websocket.test.ts

DEPENDENCIES: @modelcontextprotocol/sdk, better-sqlite3, express, express-rate-limit, ws, cors, uuid, zod

---

SECURITY

CORS: Origin whitelist (localhost:5173, 127.0.0.1:5173, localhost:3099, localhost:8012)
RATE_LIMITING: Tiered limits - General: 100/min, Routing: 30/min
HEADERS: RateLimit-Limit, RateLimit-Remaining, RateLimit-Reset (draft-7 standard)
