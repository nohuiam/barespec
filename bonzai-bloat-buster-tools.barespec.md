TOOLS: bonzai-bloat-buster
VERSION: 3.0
UPDATED: 2025-12-25
COUNT: 6

---

TOOL: analyze_directory
INPUT: { directoryPath: string (required), generatePlan?: boolean (default: true) }
OUTPUT: { files_analyzed: number, duplicates_found: number, consolidation_plan: object }
USE: Scan directory for duplicate/similar markdown documents
EXAMPLE: analyze_directory({ directoryPath: "/Users/macbook/Documents/inbox" })
NOTES: First step before any consolidation. Always generates plan unless disabled.

---

TOOL: process_documents
INPUT: { filePaths: string[] (required) }
OUTPUT: { processed_count: number, vectors_created: number }
USE: Add specific documents to vector database for future comparisons
EXAMPLE: process_documents({ filePaths: ["/path/doc1.md", "/path/doc2.md"] })
NOTES: Use when adding new documents to the index without full directory scan.

---

TOOL: compare_documents
INPUT: { filePathA: string (required), filePathB: string (required) }
OUTPUT: { semantic_similarity: 0-1, structural_similarity: 0-1, exact_similarity: 0-1, classification: "duplicate"|"near_duplicate"|"related"|"different" }
USE: Direct comparison of two specific documents
EXAMPLE: compare_documents({ filePathA: "/path/a.md", filePathB: "/path/b.md" })
NOTES: Returns three similarity scores. Classification based on semantic_similarity threshold.

---

TOOL: get_storage_stats
INPUT: {}
OUTPUT: { collection_info: object, vector_count: number, cache_stats: object }
USE: Check vector database health and statistics
EXAMPLE: get_storage_stats({})
NOTES: No input required. Use for monitoring Qdrant health.

---

TOOL: execute_consolidation_plan
INPUT: { planPath: string (required), dryRun?: boolean (default: true) }
OUTPUT: { actions_taken: number, files_merged: number, files_archived: number, backup_location: string }
USE: Execute consolidation plan with safety backups
EXAMPLE: execute_consolidation_plan({ planPath: "/path/plan.json", dryRun: true })
NOTES: ALWAYS use dryRun=true first. Creates backup before any destructive action.

---

TOOL: batch_compare_documents
INPUT: { newFiles: string[] (required), existingFiles?: string[], threshold?: number (default: 0.70), returnOnlyHighSimilarity?: boolean (default: true) }
OUTPUT: { comparisons: [{ file: string, matches: [{ path: string, similarity: number }] }] }
USE: High performance batch comparison (50x faster than sequential)
EXAMPLE: batch_compare_documents({ newFiles: ["/path/new1.md", "/path/new2.md"], threshold: 0.80 })
NOTES: Use for bulk duplicate detection. Threshold 0.70 catches related content.

---

THRESHOLDS:
- >= 0.92: Exact duplicate
- 0.80-0.92: Near duplicate (same info, different wording)
- 0.70-0.80: Related content
- < 0.70: Different documents
