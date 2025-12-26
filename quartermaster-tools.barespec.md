TOOLS: quartermaster
VERSION: 2.1
UPDATED: 2025-12-25
COUNT: 23

---

TOOL: quartermaster_search
INPUT: { query: string (required), scope?: "all"|"classified"|"unclassified"|"recent"|"stale", path?: string, file_types?: string[], limit?: number (default: 50), offset?: number }
OUTPUT: { results: [{ file_id, path, filename, score, glec_class, subject }], total: number, search_time_ms: number }
USE: Advanced FTS5 search with specialized syntax
EXAMPLE: quartermaster_search({ query: "glec:Genesis subject:Architecture", limit: 20 })
SYNTAX: dewey:, glec:, subject:, duplicate:, path:

---

TOOL: quartermaster_live_find
INPUT: { query: string (required), path?: string, max_depth?: number, file_types?: string[] }
OUTPUT: { results: array, search_time_ms: number, source: "filesystem" }
USE: Real-time filesystem search bypassing index (for very recent files)
EXAMPLE: quartermaster_live_find({ query: "*.md", path: "/Dropository" })
NOTES: Slower than index search but always current.

---

TOOL: quartermaster_hybrid_find
INPUT: { query: string (required), path?: string, file_types?: string[], prefer?: "index"|"live"|"both" }
OUTPUT: { results: array, index_results: number, live_results: number, merged_count: number }
USE: Smart hybrid search combining index + live search (RECOMMENDED)
EXAMPLE: quartermaster_hybrid_find({ query: "workflow", prefer: "both" })
NOTES: Best for most searches. Merges and deduplicates results.

---

TOOL: quartermaster_recent
INPUT: { type?: "created"|"modified"|"deleted", hours?: number (default: 24), limit?: number (default: 50) }
OUTPUT: { files: [{ path, filename, timestamp, type }], count: number }
USE: Get recently created/modified/deleted files
EXAMPLE: quartermaster_recent({ type: "modified", hours: 6 })

---

TOOL: quartermaster_inventory
INPUT: { path: string (required), options?: { include_hidden: boolean, max_depth: number } }
OUTPUT: { path: string, total_files: number, total_size_mb: number, file_types: object }
USE: Comprehensive directory analysis with statistics
EXAMPLE: quartermaster_inventory({ path: "/Users/macbook/Documents" })

---

TOOL: quartermaster_audit
INPUT: {}
OUTPUT: { audit_id: string, timestamp: number, status: "healthy"|"warning"|"critical", score: number, total_files: number, indexed_files: number, stale_files: number, unclassified_files: number, recommendations: array }
USE: Full project health assessment
EXAMPLE: quartermaster_audit({})

---

TOOL: quartermaster_get_metadata
INPUT: { filepath: string (required) }
OUTPUT: { file_id, path, filename, hash, size, created, modified, glec_class, glec_confidence, dewey_id, subject, approval_state, move_history: array }
USE: Get detailed file metadata including classification and move history
EXAMPLE: quartermaster_get_metadata({ filepath: "/path/to/file.md" })

---

TOOL: quartermaster_suggest_location
INPUT: { filepath: string (required), content?: string }
OUTPUT: { suggestions: array, classification: string, classification_confidence: number, similar_files: array, primary_suggestion: string }
USE: AI-powered file placement suggestions
EXAMPLE: quartermaster_suggest_location({ filepath: "/Dropository/new-doc.md" })

---

TOOL: quartermaster_suggest_scope
INPUT: { query: string (required) }
OUTPUT: { recommended_scope: string, reason: string, alternative_scopes: array, suggested_paths: array, file_types: array, keywords: array, estimated_results: number }
USE: Suggest optimal search scope based on query
EXAMPLE: quartermaster_suggest_scope({ query: "authentication flow" })

---

TOOL: quartermaster_inbox_drop
INPUT: { filepath: string (required) }
OUTPUT: { success: boolean, status: "queued"|"error"|"unavailable", message: string }
USE: Drop file in inbox for automated classification
EXAMPLE: quartermaster_inbox_drop({ filepath: "/Dropository/incoming.md" })

---

TOOL: quartermaster_inbox_status
INPUT: {}
OUTPUT: { total_files: number, pending_files: number, processed_files: number, auto_moved_files: number, average_confidence: number, items: array }
USE: Get inbox processing status and queue
EXAMPLE: quartermaster_inbox_status({})

---

TOOL: quartermaster_watch_status
INPUT: {}
OUTPUT: { status: "active"|"stopped"|"unavailable", watched_directories: array, files_discovered: number, files_changed: number, files_deleted: number, errors: number, uptime_seconds: number }
USE: File watcher health and status
EXAMPLE: quartermaster_watch_status({})

---

TOOL: quartermaster_approval_state
INPUT: { state?: "pending"|"approved"|"rejected"|"auto_approved", limit?: number }
OUTPUT: { files: array, total: number, by_state: object }
USE: Query files by approval status
EXAMPLE: quartermaster_approval_state({ state: "pending" })

---

TOOL: quartermaster_health
INPUT: {}
OUTPUT: { status: "healthy"|"degraded"|"critical", score: number, checks: { database, memory, watcher, indexer, alerts, inbox }, uptime_seconds: number }
USE: Comprehensive health check and diagnostics
EXAMPLE: quartermaster_health({})

---

TOOL: quartermaster_duplicates
INPUT: { path?: string, min_size?: number, include_analysis?: boolean (default: true) }
OUTPUT: { duplicate_groups: [{ hash, count, size_each_mb, total_size_mb, savings_mb, files: array }], total_groups: number, total_duplicates: number, total_savings_mb: number, analysis: object }
USE: Find duplicate files using hash comparison
EXAMPLE: quartermaster_duplicates({ path: "/Documents", min_size: 1024 })

---

TOOL: quartermaster_alerts
INPUT: { severity?: "info"|"warning"|"error"|"critical"|"all" (default: "all"), category?: string, unresolved_only?: boolean (default: true), max_results?: number (default: 50) }
OUTPUT: { alerts: [{ id, severity, category, message, remediation, alert_data }], total_alerts: number, stats: object }
USE: Get system alerts (7 categories, 4 severities)
EXAMPLE: quartermaster_alerts({ severity: "error", unresolved_only: true })

---

TOOL: quartermaster_refresh_index
INPUT: { path?: string, force_refresh?: boolean (default: false) }
OUTPUT: { status: "success"|"error"|"unavailable", files_scanned: number, message: string }
USE: Manual index refresh with progress reporting
EXAMPLE: quartermaster_refresh_index({ path: "/Documents", force_refresh: true })

---

TOOL: quartermaster_snapshot
INPUT: {}
OUTPUT: { snapshot_id: string, timestamp: string, statistics: object }
USE: Generate filesystem snapshot for audit
EXAMPLE: quartermaster_snapshot({})

---

TOOL: quartermaster_about
INPUT: {}
OUTPUT: { name: string, version: string, description: string, files_managed: number, status: string }
USE: Get server information and statistics
EXAMPLE: quartermaster_about({})

---

TOOL: quartermaster_daily_digest
INPUT: { date?: string }
OUTPUT: { date: string, file_changes: object }
USE: Daily activity summary
EXAMPLE: quartermaster_daily_digest({})

---

TOOL: quartermaster_weekly_report
INPUT: { week_start?: string }
OUTPUT: { week_start: string, summary: object }
USE: Weekly trend analysis
EXAMPLE: quartermaster_weekly_report({})

---

TOOL: reindex_files
INPUT: { path?: string, force_refresh?: boolean }
OUTPUT: { status: string, files_scanned: number, message: string }
USE: Legacy alias for quartermaster_refresh_index
EXAMPLE: reindex_files({ force_refresh: true })

---

TOOL: quartermaster_find
INPUT: { query: string (required), scope?: string, path?: string, file_types?: string[], limit?: number, offset?: number }
OUTPUT: { results: array, total: number, search_time_ms: number }
USE: Legacy alias for quartermaster_search
EXAMPLE: quartermaster_find({ query: "architecture" })

---

CATEGORIES: Search & Discovery (5), Inventory & Analysis (3), Suggestions (2), Inbox (2), Monitoring (5), Maintenance (3), Reporting (3)
