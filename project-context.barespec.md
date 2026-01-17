SERVER: project-context
VERSION: 1.4
UPDATED: 2026-01-17
STATUS: Production
PORT: 3016 (UDP/InterLock), 8016 (HTTP), 9016 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: Configuration management, path safety, and rate limiting
CONFIG: /repo/project-context/config/interlock.json

---

ARCHITECTURE

WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
FUNCTION: Prevent "Catasort the Universe" scenarios
PROTECTION: Path whitelisting, file count limits, cost ceilings, rate limiting
HISTORY: Configuration changes tracked for rollback
INTERLOCK: BaNano protocol, signals 0xA0-0xAF (CONFIG_REQUEST, CONFIG_UPDATED)

---

TOOLS (5)

TOOL: get_configuration
INPUT: { section?: "paths"|"limits"|"rate_limits"|"safety" }
OUTPUT: { paths: object, limits: object, rate_limits: object, safety: object }
USE: Get complete or section-specific project configuration
EXAMPLE: get_configuration({ section: "paths" })
NOTES: Omit section to get full configuration. Returns current active settings.

TOOL: get_path_configuration
INPUT: { path: string (required) }
OUTPUT: { allowed_operations: string[], limits: object, permissions: object, is_protected: boolean }
USE: Check what operations are allowed for a specific path
EXAMPLE: get_path_configuration({ path: "/Users/macbook/Documents/important" })
NOTES: Use before file operations to verify permissions. Protected paths have restricted operations.

TOOL: update_configuration
INPUT: { section: "paths"|"limits"|"rate_limits"|"safety" (required), settings: object (required) }
OUTPUT: { success: boolean, updated_section: string, validation_result: object }
USE: Update configuration settings (validated before applying)
EXAMPLE: update_configuration({ section: "limits", settings: { max_file_size: 10485760 } })
NOTES: Settings are validated before applying. Invalid settings return validation errors.

TOOL: validate_path
INPUT: { path: string (required), operation: "read"|"write"|"delete" (required) }
OUTPUT: { allowed: boolean, reason: string, restrictions: string[] }
USE: Check if a path is allowed for a specific operation
EXAMPLE: validate_path({ path: "/etc/passwd", operation: "read" })
NOTES: Call before any file operation. Returns reason if denied.

TOOL: check_rate_limit
INPUT: { operation: string (required), cost?: number }
OUTPUT: { allowed: boolean, remaining: number, reset_time: timestamp, cost_tracked: number }
USE: Check if operation is within rate limits, records the operation
EXAMPLE: check_rate_limit({ operation: "api_call", cost: 0.01 })
NOTES: Also records the operation. Cost is optional for tracking API spend.

---

CONFIGURATION SECTIONS

- paths: Whitelist, blacklist, max_depth
- limits: max_files_per_operation, max_file_size_mb, max_total_size_gb, max_api_calls_per_minute
- rate_limits: Per-operation limits and windows
- safety: require_confirmation_above, auto_reject_above, quarantine_suspicious

---

SAFETY: Validates paths before operations
ENFORCEMENT: Rate limits, cost tracking

---

HTTP REST API (port 8016)

ENDPOINT: GET /health
OUTPUT: { status: "healthy", server: string, version: string, uptime: object, ports: object, timestamp: string }
USE: Health check and server status

ENDPOINT: GET /stats
OUTPUT: { success: boolean, stats: { rateLimits: object, costTracking: object, paths: object, limits: object }, timestamp: string }
USE: Server statistics including rate limit and cost tracking

ENDPOINT: GET /api/v1/configuration
OUTPUT: { success: boolean, configuration: object, metadata: object }
USE: Get complete project configuration

ENDPOINT: GET /api/v1/configuration/:section
PARAMS: section = paths|limits|rate_limits|safety
OUTPUT: { success: boolean, section: string, configuration: object, timestamp: string }
USE: Get specific configuration section

ENDPOINT: POST /api/v1/paths/validate
INPUT: { path: string, operation: "read"|"write"|"delete" }
OUTPUT: { success: boolean, allowed: boolean, reason?: string, path: string, operation: string, limits: object, timestamp: string }
USE: Validate path for specific operation

ENDPOINT: GET /api/v1/paths/config
PARAMS: ?path=string
OUTPUT: { success: boolean, path: string, allowed: boolean, permissions: object, limits: object, safetyLimits: object, timestamp: string }
USE: Get configuration for specific path

ENDPOINT: POST /api/v1/rate-limits/check
INPUT: { operation: string, cost?: number }
OUTPUT: { success: boolean, allowed: boolean, remaining: number, resetIn: number, costTracking: object, timestamp: string }
USE: Check and record operation against rate limits

ENDPOINT: GET /api/v1/rate-limits/stats
OUTPUT: { success: boolean, stats: object, costTracking: object, timestamp: string }
USE: Get rate limit statistics

ENDPOINT: GET /api/tools
OUTPUT: { tools: [{ name, description, inputSchema }], count: number }
USE: List all MCP tools for bop-gateway integration

ENDPOINT: POST /api/tools/:toolName
INPUT: { arguments: object } or direct args object
OUTPUT: { success: boolean, result: object } or { success: false, error: string }
USE: Execute MCP tool via HTTP for bop-gateway integration

---

WEBSOCKET EVENTS (port 9016)

EVENT: connected
DIRECTION: server -> client
DATA: { clientId: string, server: string, version: string, availableEvents: array }
USE: Sent on connection establishment

EVENT: config_updated
DIRECTION: server -> client
DATA: { section: string, previousValue: object, newValue: object }
USE: Broadcast when configuration changes

EVENT: path_validation
DIRECTION: server -> client
DATA: { path: string, operation: string, allowed: boolean, reason?: string }
USE: Broadcast on path validation requests

EVENT: rate_limit_exceeded
DIRECTION: server -> client
DATA: { operation: string, remaining: number, resetIn: number, currentCount: number, limit: number }
USE: Broadcast when rate limit is hit

EVENT: stats_update
DIRECTION: server -> client
DATA: { rateLimits: object, currentHourCost: number }
USE: Periodic stats update

CLIENT MESSAGE: subscribe
DATA: { type: "subscribe", events: string[] }
USE: Subscribe to specific events

CLIENT MESSAGE: ping
DATA: { type: "ping" }
RESPONSE: { type: "pong", data: { timestamp: number } }
USE: Keepalive check

---

KEY FILES

SOURCE: /repo/project-context/
INDEX: src/index.ts
TYPES: src/types.ts
VALIDATION: src/validation/paths.ts, src/validation/limits.ts
HTTP: src/http/server.ts
WEBSOCKET: src/websocket/server.ts
UTILS: src/utils/logger.ts
TESTS: tests/validation.test.ts, tests/ratelimit.test.ts, tests/http.test.ts, tests/websocket.test.ts

DEPENDENCIES: @modelcontextprotocol/sdk, express, ws, cors, winston, lru-cache, zod
