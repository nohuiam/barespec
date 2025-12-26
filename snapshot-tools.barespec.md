TOOLS: snapshot
VERSION: 1.0
UPDATED: 2025-12-25
COUNT: 6

---

TOOL: snapshot_capture_snapshot
INPUT: { type?: "full"|"incremental"|"config" (default: "full"), label?: string }
OUTPUT: { snapshot_id: number, type: string, label: string, timestamp: string, file_count: number, total_size_bytes: number, checksums: object }
USE: Manually trigger snapshot capture of current system state
EXAMPLE: snapshot_capture_snapshot({ type: "full", label: "pre-migration-backup" })
NOTES: "full" captures everything. "incremental" only changes since last. "config" only configuration files.

---

TOOL: snapshot_verify_integrity
INPUT: { snapshot_id?: number }
OUTPUT: { verified: boolean, snapshot_id: number, mismatches: array, corrupted_files: array, missing_files: array, timestamp: string }
USE: Verify current system state against snapshot
EXAMPLE: snapshot_verify_integrity({ snapshot_id: 42 })
NOTES: Uses latest snapshot if id not specified. Lists all discrepancies.

---

TOOL: snapshot_list_snapshots
INPUT: { limit?: number (default: 50), type?: "full"|"incremental"|"config", status?: "captured"|"verified"|"corrupted" }
OUTPUT: { snapshots: [{ id: number, type: string, label: string, status: string, timestamp: string, file_count: number }], total: number }
USE: List all captured snapshots with optional filtering
EXAMPLE: snapshot_list_snapshots({ limit: 10, type: "full" })

---

TOOL: snapshot_get_snapshot_details
INPUT: { snapshot_id: number (required), include_files?: boolean (default: false), include_verifications?: boolean (default: true) }
OUTPUT: { id: number, type: string, label: string, status: string, timestamp: string, file_count: number, total_size: number, files?: array, verifications: array }
USE: Get detailed information about a specific snapshot
EXAMPLE: snapshot_get_snapshot_details({ snapshot_id: 42, include_files: false })
NOTES: include_files can return very large response.

---

TOOL: snapshot_compare_snapshots
INPUT: { snapshot_id_1: number (required), snapshot_id_2: number (required) }
OUTPUT: { comparison: { added: array, removed: array, modified: array, unchanged_count: number }, summary: object }
USE: Compare two snapshots to identify changes
EXAMPLE: snapshot_compare_snapshots({ snapshot_id_1: 40, snapshot_id_2: 42 })
NOTES: snapshot_id_1 should be earlier than snapshot_id_2.

---

TOOL: snapshot_restore_snapshot
INPUT: { snapshot_id: number (required), confirm: boolean (required), create_backup?: boolean (default: true) }
OUTPUT: { success: boolean, snapshot_id: number, files_restored: number, backup_created: boolean, backup_id: number, timestamp: string }
USE: Restore system state from snapshot (DESTRUCTIVE)
EXAMPLE: snapshot_restore_snapshot({ snapshot_id: 42, confirm: true, create_backup: true })
NOTES: REQUIRES confirm: true. Creates backup before restore by default.

---

TYPES: full (complete), incremental (changes), config (configuration)
STATUS: captured, verified, corrupted
VERIFICATION: SHA-256 checksums, file presence, size matching
