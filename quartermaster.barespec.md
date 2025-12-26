SERVER: quartermaster
VERSION: 2.1
UPDATED: 2025-12-26
STATUS: Production
PORT: 3002 (UDP/InterLock), 8002 (HTTP), 9002 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: Redis 6379 (optional)
PURPOSE: Filesystem lifecycle management with FTS5 search and InterLock mesh
CONFIG: /repo/Quartermaster/quartermaster/config/default.json

---

ARCHITECTURE

DATABASE: SQLite with FTS5 full-text search
FEATURES: InterLock mesh, file watcher, Langfuse monitoring, inbox processor, alert system

TOOL: quartermaster_search
INPUT: { query: string (required), scope?: "all"|"classified"|"unclassified"|"recent"|"stale", path?: string, file_types?: string[], limit?: number (default: 50), offset?: number }
OUTPUT: { results: [{ file_id, path, filename, score, glec_class, subject }], total, search_time_ms }
USE: Advanced FTS5 search with specialized syntax (dewey:, glec:, subject:, duplicate:)
EXAMPLE: quartermaster_search({ query: "glec:Genesis subject:Architecture", limit: 20 })

TOOL: quartermaster_live_find
INPUT: { query: string (required), path?: string, max_depth?: number, file_types?: string[] }
OUTPUT: { results: [], search_time_ms, source: "filesystem" }
USE: Real-time filesystem search bypassing index (for very recent files)
EXAMPLE: quartermaster_live_find({ query: "*.md", path: "/Dropository" })

TOOL: quartermaster_hybrid_find
INPUT: { query: string (required), path?: string, file_types?: string[], prefer?: "index"|"live"|"both" }
OUTPUT: { results: [], index_results, live_results, merged_count }
USE: Smart hybrid search combining index + live search (RECOMMENDED for most searches)
EXAMPLE: quartermaster_hybrid_find({ query: "workflow", prefer: "both" })

TOOL: quartermaster_recent
INPUT: { type?: "created"|"modified"|"deleted", hours?: number (default: 24), limit?: number (default: 50) }
OUTPUT: { files: [{ path, filename, timestamp, type }], count }
USE: Get recently created/modified/deleted files
EXAMPLE: quartermaster_recent({ type: "modified", hours: 6 })

TOOL: quartermaster_inventory
INPUT: { path: string (required), options?: { include_hidden: boolean, max_depth: number } }
OUTPUT: { path, total_files, total_size_mb, file_types: {} }
USE: Comprehensive directory analysis with statistics
EXAMPLE: quartermaster_inventory({ path: "/Users/macbook/Documents" })

TOOL: quartermaster_audit
INPUT: {}
OUTPUT: { audit_id, timestamp, status: "healthy"|"warning"|"critical", score, total_files, indexed_files, stale_files, unclassified_files, recommendations: [] }
USE: Full project health assessment and audit
EXAMPLE: quartermaster_audit({})

TOOL: quartermaster_get_metadata
INPUT: { filepath: string (required) }
OUTPUT: { file_id, path, filename, hash, size, created, modified, glec_class, glec_confidence, dewey_id, subject, approval_state, move_history: [] }
USE: Get detailed file metadata including classification and move history
EXAMPLE: quartermaster_get_metadata({ filepath: "/path/to/file.md" })

TOOL: quartermaster_suggest_location
INPUT: { filepath: string (required), content?: string }
OUTPUT: { suggestions: [], classification, classification_confidence, similar_files: [], primary_suggestion }
USE: AI-powered file placement suggestions with confidence scoring
EXAMPLE: quartermaster_suggest_location({ filepath: "/Dropository/new-doc.md" })

TOOL: quartermaster_suggest_scope
INPUT: { query: string (required) }
OUTPUT: { recommended_scope, reason, alternative_scopes: [], suggested_paths: [], file_types: [], keywords: [], estimated_results }
USE: Suggest optimal search scope based on query
EXAMPLE: quartermaster_suggest_scope({ query: "authentication flow" })

TOOL: quartermaster_inbox_drop
INPUT: { filepath: string (required) }
OUTPUT: { success: boolean, status: "queued"|"error"|"unavailable", message }
USE: Drop file in inbox for automated classification and placement
EXAMPLE: quartermaster_inbox_drop({ filepath: "/Dropository/incoming.md" })

TOOL: quartermaster_inbox_status
INPUT: {}
OUTPUT: { total_files, pending_files, processed_files, auto_moved_files, average_confidence, items: [] }
USE: Get inbox processing status and queue
EXAMPLE: quartermaster_inbox_status({})

TOOL: quartermaster_watch_status
INPUT: {}
OUTPUT: { status: "active"|"stopped"|"unavailable", watched_directories: [], files_discovered, files_changed, files_deleted, errors, uptime_seconds }
USE: File watcher health and status
EXAMPLE: quartermaster_watch_status({})

TOOL: quartermaster_approval_state
INPUT: { state?: "pending"|"approved"|"rejected"|"auto_approved", limit?: number }
OUTPUT: { files: [], total, by_state: {} }
USE: Query files by approval status
EXAMPLE: quartermaster_approval_state({ state: "pending" })

TOOL: quartermaster_health
INPUT: {}
OUTPUT: { status: "healthy"|"degraded"|"critical", score, checks: { database, memory, watcher, indexer, alerts, inbox }, uptime_seconds }
USE: Comprehensive health check and diagnostics
EXAMPLE: quartermaster_health({})

TOOL: quartermaster_duplicates
INPUT: { path?: string, min_size?: number, include_analysis?: boolean (default: true) }
OUTPUT: { duplicate_groups: [{ hash, count, size_each_mb, total_size_mb, savings_mb, files: [] }], total_groups, total_duplicates, total_savings_mb, analysis }
USE: Find duplicate files using hash comparison
EXAMPLE: quartermaster_duplicates({ path: "/Documents", min_size: 1024 })

TOOL: quartermaster_alerts
INPUT: { severity?: "info"|"warning"|"error"|"critical"|"all" (default: "all"), category?: string, unresolved_only?: boolean (default: true), max_results?: number (default: 50) }
OUTPUT: { alerts: [{ id, severity, category, message, remediation, alert_data }], total_alerts, stats }
USE: Get system alerts (7 categories, 4 severities)
EXAMPLE: quartermaster_alerts({ severity: "error", unresolved_only: true })

TOOL: quartermaster_refresh_index
INPUT: { path?: string, force_refresh?: boolean (default: false) }
OUTPUT: { status: "success"|"error"|"unavailable", files_scanned, message }
USE: Manual index refresh with progress reporting
EXAMPLE: quartermaster_refresh_index({ path: "/Documents", force_refresh: true })

TOOL: quartermaster_snapshot
INPUT: {}
OUTPUT: { snapshot_id, timestamp, statistics: {} }
USE: Generate filesystem snapshot for audit purposes
EXAMPLE: quartermaster_snapshot({})

TOOL: quartermaster_about
INPUT: {}
OUTPUT: { name, version, description, files_managed, status }
USE: Get server information and statistics
EXAMPLE: quartermaster_about({})

TOOL: quartermaster_daily_digest
INPUT: { date?: string }
OUTPUT: { date, file_changes: {} }
USE: Daily activity summary
EXAMPLE: quartermaster_daily_digest({})

TOOL: quartermaster_weekly_report
INPUT: { week_start?: string }
OUTPUT: { week_start, summary: {} }
USE: Weekly trend analysis
EXAMPLE: quartermaster_weekly_report({})

TOOL: reindex_files
INPUT: { path?: string, force_refresh?: boolean }
OUTPUT: { status, files_scanned, message }
USE: Legacy alias for quartermaster_refresh_index
EXAMPLE: reindex_files({ force_refresh: true })

---

TOOL CATEGORIES

- Search & Discovery (5): search, live_find, hybrid_find, recent, suggest_scope
- Inventory & Analysis (3): inventory, audit, duplicates
- Suggestions (2): suggest_location, suggest_scope
- Inbox (2): inbox_drop, inbox_status
- Monitoring (5): watch_status, approval_state, health, alerts, snapshot
- Maintenance (3): refresh_index, reindex_files, get_metadata
- Reporting (3): about, daily_digest, weekly_report

TOTAL TOOLS: 23

---

KEY FILES

SOURCE: /repo/Quartermaster/quartermaster/
NOTE: Implementation in nested directory structure
