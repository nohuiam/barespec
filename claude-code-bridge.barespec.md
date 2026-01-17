SERVER: claude-code-bridge
VERSION: 0.1.1
UPDATED: 2026-01-17
STATUS: Production
PORT: 3013 (UDP/InterLock), 8013 (HTTP), 9013 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: better-sqlite3
PURPOSE: Desktop-to-Code communication bridge with work order management
CONFIG: /repo/ClaudeCodeBridge/config/interlock.json

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3) with WAL mode
WORKFLOW: Claude Desktop creates work orders → Bridge queues → Claude Code executes → Reports back
BRIDGE: Manages bidirectional communication between Claude Desktop and Claude Code instances
OPERATIONS: catasort, batch_classify, file_move, script_execution, custom
PRIORITY: emergency > high > normal > low

---

TOOLS (8)

TOOL: create_work_order
INPUT: { sessionId: string (required), operation: "catasort"|"batch_classify"|"file_move"|"script_execution"|"custom" (required), targetPath: string (required), fileCount: number (required), useLLM?: boolean (default: true), priority?: "low"|"normal"|"high"|"emergency" (default: "normal"), instructions?: string, files?: array }
OUTPUT: { workOrderId: string, status: string, createdAt: string, estimatedCost: number, estimatedCompletion: string, safetyValidation: { passed: boolean, warnings: [] } }
USE: Create a new work order for Claude Code to execute
EXAMPLE: create_work_order({ sessionId: "sess-001", operation: "catasort", targetPath: "/Dropository", fileCount: 50 })
NOTES: Work order ID format: WO-YYYY-MM-DD-NNN. Cost estimation based on fileCount and useLLM flag.

TOOL: get_work_order
INPUT: { id: string (required) }
OUTPUT: { id: string, status: string, operation: string, targetPath: string, fileCount: number, priority: string, progress: object, createdAt: string, startedAt: string, completedAt: string, estimatedCompletion: string, results: object }
USE: Retrieve a specific work order by ID
EXAMPLE: get_work_order({ id: "WO-2025-12-27-001" })

TOOL: list_work_orders
INPUT: { status?: "pending"|"in_progress"|"completed"|"failed"|"cancelled" }
OUTPUT: { total: number, filtered: number, workOrders: [{ id, status, operation, priority, fileCount, createdAt }] }
USE: List all work orders, optionally filtered by status
EXAMPLE: list_work_orders({ status: "pending" })
NOTES: Results sorted by priority (emergency first) then by creation date.

TOOL: update_work_order
INPUT: { id: string (required), status: "in_progress"|"completed"|"failed"|"cancelled" (required), results?: object }
OUTPUT: { success: boolean, workOrder: { id, status, completedAt, results } }
USE: Update work order status (Claude Code reports progress)
EXAMPLE: update_work_order({ id: "WO-2025-12-27-001", status: "completed", results: { filesProcessed: 50 } })

TOOL: cancel_work_order
INPUT: { id: string (required) }
OUTPUT: { success: boolean, workOrder: { id, status, cancelledAt } }
USE: Cancel a pending or in-progress work order
EXAMPLE: cancel_work_order({ id: "WO-2025-12-27-001" })

TOOL: get_bridge_status
INPUT: {}
OUTPUT: { status: string, activeWorkOrders: number, pendingWorkOrders: number, completedToday: number, failedToday: number, uptime: number }
USE: Get overall bridge health and activity status
EXAMPLE: get_bridge_status({})

TOOL: get_neural_metrics
INPUT: { timeWindow?: "hour"|"day"|"week"|"all" (default: "day") }
OUTPUT: { timeWindow: string, metrics: { transmissions, latency, successRate, avgProcessingTime }, byOperation: object, byPriority: object }
USE: Get synaptic transmission metrics and performance stats
EXAMPLE: get_neural_metrics({ timeWindow: "week" })

TOOL: prune_old_work_orders
INPUT: { olderThanDays?: number (default: 7) }
OUTPUT: { success: boolean, archived: { count, oldestDate, newestDate }, archiveLocation: string }
USE: Archive completed/failed work orders older than N days
EXAMPLE: prune_old_work_orders({ olderThanDays: 30 })

---

LAYERS

1. MCP stdio (8 tools) - Primary interface
2. InterLock UDP mesh (port 3013) - Peer communication
3. HTTP REST API (port 8013) - External integrations
4. WebSocket real-time (port 9013) - Live updates

---

HTTP REST API (port 8013)

ENDPOINT: GET /health
OUTPUT: { status, server, uptime }
USE: Health check

ENDPOINT: GET /api/tools
OUTPUT: { tools: [{ name, description, inputSchema }], count: number }
USE: List all MCP tools for bop-gateway integration

ENDPOINT: POST /api/tools/:toolName
INPUT: { arguments: object } or direct args object
OUTPUT: { success: boolean, result: object } or { success: false, error: string }
USE: Execute MCP tool via HTTP for bop-gateway integration

---

KEY CONCEPTS

WORK_ORDER: A task package for Claude Code with operation, target, priority
SESSION: Active Claude Code instance tracked by sessionId
BRIDGE: Communication layer between Desktop and Code instances
NEURAL_METRIC: Performance measurement (latency, success rate, throughput)
RELAY_LOG: Record of inter-server communication

---

INTERLOCK SIGNALS

Accepts: DOCK_REQUEST (0x01), HEARTBEAT (0x04), WORK_ORDER_CREATED (0xC3), WORK_ORDER_UPDATED (0xC4), WORK_ORDER_COMPLETED (0xC5), BUILD_PROGRESS (0xC6), RELAY_REQUEST (0xC7)
Connections: trinity-coordinator (3012), chronos (3011), context-guardian (3001)

---

DATABASE SCHEMA

TABLE: work_orders
COLUMNS: id, session_id, operation, target_path, file_count, use_llm, priority, status, instructions, files, results, created_at, started_at, completed_at, estimated_cost, actual_cost
INDEXES: status, priority, session_id, created_at

TABLE: code_sessions
COLUMNS: id, started_at, last_activity, status, context
INDEXES: status

TABLE: build_progress
COLUMNS: id, work_order_id, build_type, current_step, total_steps, percentage, status, started_at, completed_at, error_log
INDEXES: work_order_id

TABLE: relay_log
COLUMNS: id, work_order_id, target_server, request_type, request_data, response_data, timestamp, latency_ms, success
INDEXES: work_order_id, target_server

TABLE: neural_metrics
COLUMNS: id, metric_type, work_order_id, value, unit, timestamp
INDEXES: timestamp

---

KEY FILES

SOURCE: /repo/ClaudeCodeBridge/
INDEX: src/index.ts (build/index.js)
TOOLS: src/tools/tool-schemas.ts
CORE: src/core/work-order-manager.ts, session-tracker.ts, build-monitor.ts, result-aggregator.ts
DATABASE: src/utils/database.ts
METRICS: src/utils/metrics.ts
VALIDATION: src/utils/validation.ts
MONITORING: src/monitoring/langfuse.ts
CONFIG: config/server.json, config/interlock.json, config/monitoring.json

DEPENDENCIES: @modelcontextprotocol/sdk, better-sqlite3, zod
