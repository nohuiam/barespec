TOOLS: pk-manager
VERSION: 1.0
UPDATED: 2025-12-25
COUNT: 4

---

TOOL: pk_rebuild
INPUT: { source_path: string (required), options?: { dry_run: boolean (default: false), skip_backup: boolean (default: false), force: boolean (default: false), verificationLevel: "basic"|"standard"|"paranoid" (default: "standard") } }
OUTPUT: { rebuildId: string, status: string, phases: array, documentsTotal: number, documentsProcessed: number, documentsSucceeded: number, documentsFailed: number, verificationResults: array, backupPath: string, duration: number }
USE: Orchestrate complete PK rebuild with 7-phase workflow
EXAMPLE: pk_rebuild({ source_path: "/Users/macbook/Documents/inbox", options: { dry_run: true } })
NOTES: ALWAYS dry_run first. "paranoid" verification runs all 6 tests twice. force=true skips pre-flight warnings.

---

TOOL: get_rebuild_status
INPUT: { rebuildId: string (required) }
OUTPUT: { success: boolean, rebuild: { id: string, status: string, phases: array, documentsTotal: number, documentsSucceeded: number } }
USE: Check status of a specific rebuild operation
EXAMPLE: get_rebuild_status({ rebuildId: "rebuild_abc123" })
NOTES: Poll during long rebuilds. Status: pending, in_progress, success, failed, rolled_back.

---

TOOL: list_rebuilds
INPUT: { limit?: number (default: 10), status?: "pending"|"in_progress"|"success"|"failed"|"rolled_back" }
OUTPUT: { success: boolean, count: number, rebuilds: [{ id: string, status: string, startedAt: timestamp, completedAt: timestamp, documentsTotal: number, documentsSucceeded: number }] }
USE: List recent rebuild operations with optional status filter
EXAMPLE: list_rebuilds({ limit: 5, status: "success" })
NOTES: Use to find rebuildId for rollback or status check.

---

TOOL: rollback_rebuild
INPUT: { rebuildId: string (required), backupId?: string }
OUTPUT: { success: boolean, message: string, backupUsed: string }
USE: Rollback to a previous backup state
EXAMPLE: rollback_rebuild({ rebuildId: "rebuild_abc123" })
NOTES: Uses latest backup if backupId not specified. Rollback is itself a tracked operation.

---

7-PHASE WORKFLOW:
1. Pre-flight Safety Checks
2. PK Evacuation & Backup
3. Document Discovery
4. Classification Pipeline
5. Knowledge Curation
6. PK Upload
7. Verification & Rollback

VERIFICATION TESTS: entry_count, checksum (SHA-256), sample_verification, glec_distribution, cross_references, search_index
GUARANTEE: 0% data loss with automatic rollback on verification failure
