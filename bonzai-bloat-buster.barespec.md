SERVER: bonzai-bloat-buster
VERSION: 3.1
UPDATED: 2025-12-26
STATUS: Production
PORT: 3008 (UDP/InterLock), 8008 (HTTP), 9008 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: Qdrant 6333 (vector DB - required)
PURPOSE: Document deduplication via vector similarity detection
CONFIG: /repo/bonzai-bloat-buster/config/interlock.json

---

ARCHITECTURE

WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
VECTOR DB: Qdrant (local)
EMBEDDINGS: all-MiniLM-L6-v2 (384 dimensions)
CONCURRENCY: p-queue with 10 parallel operations
INTERLOCK: BaNano protocol, signals 0x80-0x8F (DUPLICATE_SCAN, DUPLICATE_DETECTED, BLOAT_ANALYSIS)

---

TOOLS (6)

TOOL: analyze_directory
INPUT: { directoryPath: string (required), generatePlan?: boolean (default: true) }
OUTPUT: { files_analyzed, duplicates_found, consolidation_plan }
USE: Scan directory for duplicate/similar markdown documents
EXAMPLE: analyze_directory({ directoryPath: "/Users/macbook/Documents/inbox" })
NOTES: First step before any consolidation. Always generates plan unless disabled.

TOOL: process_documents
INPUT: { filePaths: string[] (required) }
OUTPUT: { processed_count, vectors_created }
USE: Add specific documents to vector database for future comparisons
EXAMPLE: process_documents({ filePaths: ["/path/doc1.md", "/path/doc2.md"] })
NOTES: Use when adding new documents to the index without full directory scan.

TOOL: compare_documents
INPUT: { filePathA: string (required), filePathB: string (required) }
OUTPUT: { semantic_similarity: 0-1, structural_similarity: 0-1, exact_similarity: 0-1, classification: "duplicate"|"near_duplicate"|"related"|"different" }
USE: Direct comparison of two specific documents
EXAMPLE: compare_documents({ filePathA: "/path/a.md", filePathB: "/path/b.md" })
NOTES: Returns three similarity scores. Classification based on semantic_similarity threshold.

TOOL: get_storage_stats
INPUT: {}
OUTPUT: { collection_info, vector_count, cache_stats }
USE: Check vector database health and statistics
EXAMPLE: get_storage_stats({})
NOTES: No input required. Use for monitoring Qdrant health.

TOOL: execute_consolidation_plan
INPUT: { planPath: string (required), dryRun?: boolean (default: true) }
OUTPUT: { actions_taken, files_merged, files_archived, backup_location }
USE: Execute consolidation plan with safety backups. Always use dryRun=true first.
EXAMPLE: execute_consolidation_plan({ planPath: "/path/plan.json", dryRun: true })
NOTES: ALWAYS use dryRun=true first. Creates backup before any destructive action.

TOOL: batch_compare_documents
INPUT: { newFiles: string[] (required), existingFiles?: string[], threshold?: number (default: 0.70), returnOnlyHighSimilarity?: boolean (default: true) }
OUTPUT: { comparisons: [{ file, matches: [{ path, similarity }] }] }
USE: High performance batch comparison. 50x faster than sequential.
EXAMPLE: batch_compare_documents({ newFiles: ["/path/new1.md", "/path/new2.md"], threshold: 0.80 })
NOTES: Use for bulk duplicate detection. Threshold 0.70 catches related content.

---

SIMILARITY THRESHOLDS

- >= 0.92: Exact duplicate
- 0.80-0.92: Near duplicate (same info, different wording)
- 0.70-0.80: Related content
- < 0.70: Different documents

---

KEY FILES

SOURCE: /repo/bonzai-bloat-buster/
INDEX: src/index.ts
TYPES: src/types.ts
EMBEDDING: src/embedding/embeddingPipeline.js, src/embedding/vectorStorage.js
DETECTION: src/detection/similarityDetector.js, src/detection/obsolescenceDetector.js
ANALYSIS: src/analysis/reportGenerator.js, src/analysis/consolidationPlanner.js, src/analysis/consolidationExecutor.js
PROCESSING: src/processing/documentChunker.js

DEPENDENCIES: Qdrant (local), all-MiniLM-L6-v2 embeddings (384 dimensions)
