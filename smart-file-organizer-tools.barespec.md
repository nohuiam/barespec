TOOLS: smart-file-organizer
VERSION: 2.0
UPDATED: 2025-12-25
COUNT: 3

---

TOOL: organize_files
INPUT: { sourcePath: string (required), options?: { dryRun: boolean (default: false), validateAll: boolean (default: true), batchSize: number (default: 10), confidenceThreshold: 0-1 (default: 0.3), classificationMode: "heuristic"|"catasorter"|"both" (default: "heuristic"), errorAbortThreshold: 0-1 (default: 0.2) } }
OUTPUT: { operation_id: string, files_discovered: number, files_organized: number, files_skipped: number, errors: array, movements: [{ from: string, to: string, classification: object }] }
USE: Organize files from source path using heuristic GLEC classification
EXAMPLE: organize_files({ sourcePath: "/Dropository", options: { dryRun: true } })
NOTES: Use dryRun first. "heuristic" is fast/local, "catasorter" uses external MCP, "both" combines.

---

TOOL: get_organization_stats
INPUT: { timeRange?: "hour"|"day"|"week"|"month"|"all" (default: "day"), includeReviewQueue?: boolean (default: true), includeErrorAnalysis?: boolean (default: true) }
OUTPUT: { total_operations: number, files_organized: number, success_rate: number, by_classification: object, recent_movements?: array }
USE: Get organization statistics and metrics
EXAMPLE: get_organization_stats({ timeRange: "week" })
NOTES: includeReviewQueue shows files needing manual review.

---

TOOL: rollback_operation
INPUT: { movementId?: string, batchId?: string, timeRange?: "hour"|"day"|"all", confirmDangerous?: boolean (default: false) }
OUTPUT: { success: boolean, files_restored: number, errors: array }
USE: Rollback file movements to restore files to original locations
EXAMPLE: rollback_operation({ movementId: "mov_123", confirmDangerous: true })
NOTES: Supports single file, batch, or time-range rollbacks. Requires confirmDangerous for dangerous ops.

---

CLASSIFICATION MODES:
- heuristic: Fast, local, uses file patterns and content heuristics
- catasorter: Uses CataSORTER MCP for GLEC classification
- both: Runs heuristic first, uses catasorter for validation

SERVICES:
- FileDiscoveryService: Watches /Dropository for new files
- HeuristicClassifier: Pattern-based classification
- FileMovementService: Safe file moves with audit trail
- AnalysisService: Content analysis
- ConsolidationService: Duplicate handling

SAFETY: Rollback support, operation tracking, validation
