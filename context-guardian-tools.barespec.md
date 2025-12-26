TOOLS: context-guardian
VERSION: 2.0
UPDATED: 2025-12-25
COUNT: 14

---

TOOL: context_guardian_validate
INPUT: { operation_type: string (required), context: { path?: string, server?: string, batch_size?: number, reason?: string, session_id?: string } (required), severity?: "low"|"medium"|"high"|"critical" }
OUTPUT: { approved: boolean, risk_score: number, warnings: array, recommendations: array, rules_checked: array }
USE: Validate single operation against rules, knowledge library, and session state
EXAMPLE: context_guardian_validate({ operation_type: "file_delete", context: { path: "/important/file.md" }, severity: "high" })
NOTES: Call before any risky operation. Risk score 0-100. Higher score = higher risk.

---

TOOL: context_guardian_validate_batch
INPUT: { operation_type: string (required), batch_size: number (required), sample_items?: [{ path: string, metadata?: object }], estimated_duration_seconds?: number, resource_requirements?: { memory_mb?: number, disk_mb?: number, cpu_cores?: number }, session_id?: string }
OUTPUT: { approved: boolean, risk_score: number, batch_analysis: object, warnings: array, max_recommended_batch: number }
USE: Validate batch operations with resource/risk assessment
EXAMPLE: context_guardian_validate_batch({ operation_type: "classify", batch_size: 100, estimated_duration_seconds: 300 })
NOTES: Returns max_recommended_batch if requested size too large.

---

TOOL: context_guardian_record_mistake
INPUT: { operation_type: string (required), what_happened: string (required), why_it_happened: string (required), impact: "low"|"medium"|"high"|"critical" (required), context: object (required), time_wasted_minutes?: number, recovery_steps?: string[], prevention_rule?: string }
OUTPUT: { mistake_id: string, pattern_learned: boolean, rules_generated: array, similar_mistakes: array }
USE: Record mistake with context for pattern learning
EXAMPLE: context_guardian_record_mistake({ operation_type: "file_move", what_happened: "Moved to wrong folder", why_it_happened: "Misread GLEC", impact: "medium", context: { file: "/path/file.md" } })
NOTES: Automatically generates prevention rules from patterns.

---

TOOL: context_guardian_get_stats
INPUT: { time_period?: "hour"|"day"|"week"|"month"|"all", include_trends?: boolean, metric_categories?: string[] }
OUTPUT: { validations_total: number, approvals: number, rejections: number, mistakes_recorded: number, rules_active: number, avg_risk_score: number }
USE: Get comprehensive statistics and metrics
EXAMPLE: context_guardian_get_stats({ time_period: "week" })

---

TOOL: context_guardian_get_mistakes
INPUT: { filter?: { operation_type?: string, impact?: "low"|"medium"|"high"|"critical", date_from?: string, date_to?: string }, limit?: number, include_patterns?: boolean }
OUTPUT: { mistakes: [{ id, operation_type, what_happened, impact, recorded_at }], total: number }
USE: Query mistake history with filtering
EXAMPLE: context_guardian_get_mistakes({ filter: { impact: "high" }, limit: 10 })

---

TOOL: context_guardian_session_state
INPUT: { query_type?: "files_created_recently"|"pending_operations"|"index_lag_check"|"session_summary", session_id?: string, time_window_minutes?: number }
OUTPUT: { session: { id: string, started_at: timestamp, operations: array, risk_accumulator: number, warnings: array } }
USE: Track session state and detect index lag
EXAMPLE: context_guardian_session_state({ query_type: "session_summary" })
NOTES: Risk accumulates across operations in a session.

---

TOOL: context_guardian_knowledge_lookup
INPUT: { query_type?: "is_prohibited"|"is_approved"|"get_pattern"|"get_all", query?: string, category?: string }
OUTPUT: { results: [{ topic: string, content: string, confidence: number }], total: number }
USE: Fast O(1) knowledge library lookups
EXAMPLE: context_guardian_knowledge_lookup({ query_type: "get_pattern", query: "file organization" })

---

TOOL: context_guardian_add_rule
INPUT: { rule_type: "security"|"policy"|"performance"|"custom" (required), severity: "low"|"medium"|"high"|"critical" (required), condition: object (required), action: "APPROVED"|"WARNING"|"HARD_BLOCK" (required), description: string (required), reasoning?: string, expiry_date?: string }
OUTPUT: { rule_id: string, created_at: timestamp, active: boolean }
USE: Add custom validation rule
EXAMPLE: context_guardian_add_rule({ rule_type: "security", severity: "high", condition: { path_pattern: "/protected/*" }, action: "HARD_BLOCK", description: "Block writes to protected directory" })
NOTES: Rules can have expiry dates for temporary policies.

---

TOOL: context_guardian_get_rules
INPUT: { filter?: { rule_type?: string, severity?: string, action?: string, active_only?: boolean }, limit?: number }
OUTPUT: { rules: [{ id, rule_type, condition, action, priority }], total: number }
USE: Query active validation rules
EXAMPLE: context_guardian_get_rules({ filter: { active_only: true } })

---

TOOL: context_guardian_get_guidelines
INPUT: { operation_type: string (required), context?: object }
OUTPUT: { guidelines: [{ guideline: string, priority: number, reason: string }], operation_specific: array }
USE: Get context-aware guidelines for an operation
EXAMPLE: context_guardian_get_guidelines({ operation_type: "batch_classify" })

---

TOOL: context_guardian_search_solutions
INPUT: { query: string (required), context?: object, limit?: number (default: 10) }
OUTPUT: { solutions: [{ title: string, description: string, steps: array, confidence: number }], total: number }
USE: Search past solutions for similar problems
EXAMPLE: context_guardian_search_solutions({ query: "duplicate file handling" })

---

TOOL: context_guardian_detect_drift
INPUT: { target_type?: "server_config"|"security_rules"|"file_structure"|"full_system", target?: string, baseline?: string }
OUTPUT: { drift_detected: boolean, changes: array, risk_assessment: object, recommendations: array }
USE: Detect configuration drift from baseline
EXAMPLE: context_guardian_detect_drift({ target_type: "security_rules" })
NOTES: Uses latest snapshot if baseline not specified.

---

TOOL: context_guardian_capture_snapshot
INPUT: { snapshot_type?: "server_config"|"security_rules"|"file_structure"|"full_system", targets?: string[], label?: string, set_as_baseline?: boolean }
OUTPUT: { snapshot_id: string, timestamp: string, state: object }
USE: Capture configuration snapshot for drift detection
EXAMPLE: context_guardian_capture_snapshot({ snapshot_type: "full_system", label: "pre-migration", set_as_baseline: true })

---

TOOL: context_guardian_get_validation_history
INPUT: { filter?: { operation_type?: string, status?: string, session_id?: string, date_from?: string, date_to?: string, server?: string }, limit?: number, include_stats?: boolean }
OUTPUT: { history: [{ validation_id: string, operation_type: string, approved: boolean, risk_score: number, timestamp: string, context: object }], total: number, summary: { total_validations: number, approval_rate: number, avg_risk_score: number } }
USE: Query validation history with filtering
EXAMPLE: context_guardian_get_validation_history({ filter: { operation_type: "file_move" }, limit: 20 })
NOTES: Useful for auditing and pattern analysis.

---

SERVICES: Validation, Mistakes, Session, Knowledge, Rules, Guidelines, Drift
LEARNING: Pattern detection from recorded mistakes generates prevention rules
LAYERS: MCP stdio, InterLock UDP, HTTP REST, WebSocket real-time
