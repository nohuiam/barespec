SERVER: project-context
VERSION: 1.2
UPDATED: 2025-12-27
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
SAFETY: Validates paths before operations
ENFORCEMENT: Rate limits, cost tracking
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
USE: Check if operation is within rate limits (records the operation)
EXAMPLE: check_rate_limit({ operation: "api_call", cost: 0.01 })
NOTES: Also records the operation. Cost is optional for tracking API spend.

---

CONFIGURATION SECTIONS

- paths: Whitelist, blacklist, max_depth
- limits: max_files_per_operation, max_file_size_mb, max_total_size_gb, max_api_calls_per_minute
- rate_limits: Per-operation limits and windows
- safety: require_confirmation_above, auto_reject_above, quarantine_suspicious

---

KEY FILES

SOURCE: /repo/project-context/
INDEX: src/index.ts
TYPES: src/types.ts
VALIDATION: src/validation/paths.ts, src/validation/limits.ts
UTILS: src/utils/logger.ts

DEPENDENCIES: None (standalone safety server)
