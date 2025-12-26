SERVER: context-guardian
VERSION: 2.0
UPDATED: 2025-12-26
STATUS: Production
PORT: 3001 (UDP/InterLock), 8001 (HTTP), 9001 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: Redis 6379 (vault)
PURPOSE: Security gate and mistake learning sentry for operation validation
CONFIG: /repo/imminenceV2/context-guardian/config/interlock.json

---

ARCHITECTURE

DATABASE: SQLite (better-sqlite3)
WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
SAFETY: Pattern learning from mistakes, automatic rule generation
VAULT: Redis with graceful SQLite fallback

---

TOOLS (14)

TOOL: context_guardian_validate
INPUT: { operation_type: string (required), context: { path?: string, server?: string, batch_size?: number, reason?: string, session_id?: string } (required), severity?: "low"|"medium"|"high"|"critical" }
OUTPUT: { approved: boolean, risk_score, warnings: [], recommendations: [], rules_checked: [] }
USE: Validate single operation against rules, knowledge library, and session state
EXAMPLE: context_guardian_validate({ operation_type: "file_delete", context: { path: "/important/file.md" }, severity: "high" })

TOOL: context_guardian_validate_batch
INPUT: { operation_type: string (required), batch_size: number (required), sample_items?: [{ path: string, metadata?: object }], estimated_duration_seconds?: number, resource_requirements?: { memory_mb?: number, disk_mb?: number, cpu_cores?: number }, session_id?: string }
OUTPUT: { approved: boolean, risk_score, batch_analysis: {}, warnings: [], max_recommended_batch: number }
USE: Validate batch operations with resource/risk assessment
EXAMPLE: context_guardian_validate_batch({ operation_type: "classify", batch_size: 100, estimated_duration_seconds: 300 })

TOOL: context_guardian_record_mistake
INPUT: { operation_type: string (required), what_happened: string (required), why_it_happened: string (required), impact: "low"|"medium"|"high"|"critical" (required), context: object (required), time_wasted_minutes?: number, recovery_steps?: string[], prevention_rule?: string }
OUTPUT: { mistake_id, pattern_learned: boolean, rules_generated: [], similar_mistakes: [] }
USE: Record mistake with context for pattern learning and automatic prevention rules
EXAMPLE: context_guardian_record_mistake({ operation_type: "file_move", what_happened: "Moved file to wrong folder", why_it_happened: "Misread GLEC classification", impact: "medium", context: { file: "/path/to/file.md" } })

TOOL: context_guardian_get_stats
INPUT: { time_period?: "hour"|"day"|"week"|"month"|"all", include_trends?: boolean, metric_categories?: string[] }
OUTPUT: { validations_total, approvals, rejections, mistakes_recorded, rules_active, avg_risk_score }
USE: Get comprehensive statistics and metrics
EXAMPLE: context_guardian_get_stats({ time_period: "week" })

TOOL: context_guardian_get_mistakes
INPUT: { filter?: { operation_type?: string, impact?: "low"|"medium"|"high"|"critical", date_from?: string, date_to?: string }, limit?: number, include_patterns?: boolean }
OUTPUT: { mistakes: [{ id, operation_type, what_happened, impact, recorded_at }], total }
USE: Query mistake history with filtering
EXAMPLE: context_guardian_get_mistakes({ filter: { impact: "high" }, limit: 10 })

TOOL: context_guardian_session_state
INPUT: { query_type?: "files_created_recently"|"pending_operations"|"index_lag_check"|"session_summary", session_id?: string, time_window_minutes?: number }
OUTPUT: { session: { id, started_at, operations: [], risk_accumulator, warnings: [] } }
USE: Track session state and detect index lag
EXAMPLE: context_guardian_session_state({ query_type: "session_summary" })

TOOL: context_guardian_knowledge_lookup
INPUT: { query_type?: "is_prohibited"|"is_approved"|"get_pattern"|"get_all", query?: string, category?: string }
OUTPUT: { results: [{ topic, content, confidence }], total }
USE: Fast O(1) knowledge library lookups
EXAMPLE: context_guardian_knowledge_lookup({ query_type: "get_pattern", query: "file organization" })

TOOL: context_guardian_add_rule
INPUT: { rule_type: "security"|"policy"|"performance"|"custom" (required), severity: "low"|"medium"|"high"|"critical" (required), condition: object (required), action: "APPROVED"|"WARNING"|"HARD_BLOCK" (required), description: string (required), reasoning?: string, expiry_date?: string }
OUTPUT: { rule_id, created_at, active }
USE: Add custom validation rule
EXAMPLE: context_guardian_add_rule({ rule_type: "security", severity: "high", condition: { path_pattern: "/protected/*" }, action: "HARD_BLOCK", description: "Block writes to protected directory" })

TOOL: context_guardian_get_rules
INPUT: { filter?: { rule_type?: string, severity?: string, action?: string, active_only?: boolean }, limit?: number }
OUTPUT: { rules: [{ id, rule_type, condition, action, priority }], total }
USE: Query active validation rules
EXAMPLE: context_guardian_get_rules({ filter: { active_only: true } })

TOOL: context_guardian_get_guidelines
INPUT: { operation_type: string (required), context?: object }
OUTPUT: { guidelines: [{ guideline, priority, reason }], operation_specific: [] }
USE: Get context-aware guidelines for an operation type
EXAMPLE: context_guardian_get_guidelines({ operation_type: "batch_classify" })

TOOL: context_guardian_search_solutions
INPUT: { query: string (required), context?: object, limit?: number (default: 10) }
OUTPUT: { solutions: [{ title, description, steps: [], confidence }], total }
USE: Search past solutions for similar problems
EXAMPLE: context_guardian_search_solutions({ query: "duplicate file handling" })

TOOL: context_guardian_detect_drift
INPUT: { target_type?: "server_config"|"security_rules"|"file_structure"|"full_system", target?: string, baseline?: string }
OUTPUT: { drift_detected: boolean, changes: [], risk_assessment, recommendations: [] }
USE: Detect configuration drift from baseline
EXAMPLE: context_guardian_detect_drift({ target_type: "security_rules" })

TOOL: context_guardian_capture_snapshot
INPUT: { snapshot_type?: "server_config"|"security_rules"|"file_structure"|"full_system", targets?: string[], label?: string, set_as_baseline?: boolean }
OUTPUT: { snapshot_id, timestamp, state: {} }
USE: Capture configuration snapshot for drift detection
EXAMPLE: context_guardian_capture_snapshot({ snapshot_type: "full_system", label: "pre-migration", set_as_baseline: true })

TOOL: context_guardian_get_validation_history
INPUT: { filter?: { operation_type?: string, status?: string, session_id?: string, date_from?: string, date_to?: string, server?: string }, limit?: number, include_stats?: boolean }
OUTPUT: { history: [{ validation_id, operation_type, approved, risk_score, timestamp, context }], total, summary: { total_validations, approval_rate, avg_risk_score } }
USE: Query validation history with filtering
EXAMPLE: context_guardian_get_validation_history({ filter: { operation_type: "file_move" }, limit: 20 })

---

LAYERS

1. MCP stdio (14 tools) - Primary interface
2. InterLock UDP mesh (port 3001) - Peer communication
3. HTTP REST API (port 8001) - External integrations
4. WebSocket real-time (port 9001) - Live updates

---

SERVICES

- Validation: Operation risk assessment
- Mistakes: Pattern learning from errors
- Session: Stateful tracking, index lag detection
- Knowledge: O(1) rule lookups
- Rules: Custom validation rules
- Guidelines: Context-aware guidance
- Drift: Configuration change detection

---

KEY FILES

SOURCE: /repo/imminenceV2/context-guardian/
INDEX: src/index.js
SERVICES: src/services/*.js (validation-enhanced, mistakes, session, knowledge, rules, guidelines, drift)
SECURITY: src/security/*.js (path-validator, regex-validator, error-sanitizer)
INTERLOCK: src/interlock/socket.js
HTTP: src/http/server.js
WEBSOCKET: src/websocket/server.js
VAULT: src/vault/redis-vault.js
DATABASE: src/database/client.js, schema.js
CONFIG: config/server.json, config/interlock.json

DEPENDENCIES: better-sqlite3, @modelcontextprotocol/sdk, express, ws, zod, redis (optional)
