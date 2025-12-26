TOOLS: catasorter
VERSION: 2.0
UPDATED: 2025-12-25
COUNT: 6

---

TOOL: classify_document
INPUT: { file_path: string (required), extract_nuggets?: boolean (default: true), user_hint?: string }
OUTPUT: { classification: { primary_category: string, subject: string, glec: { genesis: number, leviticus: number, exodus: number, context: number }, confidence: number, importance: 1-5, complexity: 1-5 }, dewey_id: string, gold_nuggets: array, metadata_filename: string }
USE: Classify a single document using GLEC framework
EXAMPLE: classify_document({ file_path: "/Dropository/new-doc.md", extract_nuggets: true })
NOTES: GLEC percentages sum to 100. user_hint biases classification toward expected category.

---

TOOL: get_classification
INPUT: { file_path: string (required) }
OUTPUT: { file_path: string, file_name: string, classified_at: timestamp, glec: { genesis, leviticus, exodus, context }, subject: string, dewey_id: string, confidence: number, importance: number, complexity: number, llm_method: string }
USE: Get classification for a previously classified document
EXAMPLE: get_classification({ file_path: "/Classified/Genesis/Architecture/doc.md" })
NOTES: Returns null if file not in classification database.

---

TOOL: get_dewey_stats
INPUT: {}
OUTPUT: { total_documents: number, counters: { Genesis: object, Leviticus: object, Exodus: object, Context: object }, next_ids: object }
USE: Get Dewey ID statistics and counters
EXAMPLE: get_dewey_stats({})
NOTES: next_ids shows what ID will be assigned next for each category/subject combination.

---

TOOL: get_workflow_metrics
INPUT: { include_details?: boolean (default: false) }
OUTPUT: { total_classifications: number, by_category: object, by_subject: object, by_method: object, average_confidence: number, gold_nuggets: { total: number, in_project_knowledge: number, pending_review: number } }
USE: Get classification workflow metrics and statistics
EXAMPLE: get_workflow_metrics({ include_details: true })
NOTES: by_method shows LLM vs heuristic classification breakdown.

---

TOOL: organize_document
INPUT: { file_path: string (required), user_hint?: string, dry_run?: boolean (default: false) }
OUTPUT: { success: boolean, operation: "simulated"|"completed", classification: object, original_path: string, new_path: string, folder_created: boolean, file_moved: boolean }
USE: Classify and organize document to final location
EXAMPLE: organize_document({ file_path: "/Dropository/doc.md", dry_run: true })
NOTES: Creates destination folder if needed. ALWAYS dry_run first.

---

TOOL: batch_process
INPUT: { max_files?: number (default: 10), dry_run?: boolean (default: false), extract_nuggets?: boolean (default: true) }
OUTPUT: { success: boolean, operation: "simulated"|"completed", files_discovered: number, files_processed: number, files_failed: number, results: array }
USE: Batch process documents from Dropository
EXAMPLE: batch_process({ max_files: 25, dry_run: true })
NOTES: Processes markdown and text files. Results include per-file success/error.

---

GLEC CATEGORIES:
- Genesis: Strategy, vision, concepts, architectural decisions
- Leviticus: Process, workflows, coordination, documentation
- Exodus: Implementation, code, scripts, technical execution
- Context: Reference material, external sources, background

DEWEY FORMAT: {G|L|E|C}_{Subject}_{###}
