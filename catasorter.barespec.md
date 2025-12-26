SERVER: catasorter
VERSION: 2.0
UPDATED: 2025-12-26
STATUS: Production
PORT: 3005 (UDP/InterLock), 8005 (HTTP), 9005 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: GLEC document classification and knowledge organization
CONFIG: /repo/Catasorter/config/interlock.json

---

ARCHITECTURE

CLASSIFICATION: GLEC framework (Genesis, Leviticus, Exodus, Context)
DEWEY: Hierarchical ID system for document tracking
NUGGETS: Gold nugget extraction for key insights
DATABASE: SQLite (better-sqlite3)
LAYERS: MCP stdio (6 tools), InterLock UDP mesh, HTTP REST, WebSocket events

---

TOOLS (6)

TOOL: classify_document
INPUT: { file_path: string (required), extract_nuggets?: boolean (default: true), user_hint?: string }
OUTPUT: { classification: { primary_category, subject, glec: { genesis, leviticus, exodus, context }, confidence, importance, complexity }, dewey_id, gold_nuggets: [], metadata_filename }
USE: Classify a single document using GLEC framework with optional nugget extraction
EXAMPLE: classify_document({ file_path: "/Dropository/new-doc.md", extract_nuggets: true })

TOOL: get_classification
INPUT: { file_path: string (required) }
OUTPUT: { file_path, file_name, classified_at, glec: { genesis, leviticus, exodus, context }, subject, dewey_id, confidence, importance, complexity, llm_method }
USE: Get classification for a previously classified document
EXAMPLE: get_classification({ file_path: "/Classified/Genesis/Architecture/doc.md" })

TOOL: get_dewey_stats
INPUT: {}
OUTPUT: { total_documents, counters: { Genesis: {}, Leviticus: {}, ... }, next_ids: {} }
USE: Get Dewey ID statistics and counters
EXAMPLE: get_dewey_stats({})

TOOL: get_workflow_metrics
INPUT: { include_details?: boolean (default: false) }
OUTPUT: { total_classifications, by_category: {}, by_subject: {}, by_method: {}, average_confidence, gold_nuggets: { total, in_project_knowledge, pending_review } }
USE: Get classification workflow metrics and statistics
EXAMPLE: get_workflow_metrics({ include_details: true })

TOOL: organize_document
INPUT: { file_path: string (required), user_hint?: string, dry_run?: boolean (default: false) }
OUTPUT: { success: boolean, operation, classification: {}, original_path, new_path, folder_created, file_moved }
USE: Classify and organize document to final location in Classified folder
EXAMPLE: organize_document({ file_path: "/Dropository/doc.md", dry_run: true })

TOOL: batch_process
INPUT: { max_files?: number (default: 10), dry_run?: boolean (default: false), extract_nuggets?: boolean (default: true) }
OUTPUT: { success: boolean, operation, files_discovered, files_processed, files_failed, results: [] }
USE: Batch process documents from Dropository
EXAMPLE: batch_process({ max_files: 25, dry_run: true })

---

GLEC FRAMEWORK

- Genesis: Strategy, vision, architecture, concepts
- Leviticus: Process, workflow, procedures, coordination
- Exodus: Implementation, code, execution, technical
- Context: Reference, documentation, external sources

DEWEY FORMAT: {G|L|E|C}_{Subject}_{###} (e.g., G_Architecture_001)

---

INTERLOCK SIGNALS

Emits: CLASSIFICATION_COMPLETE (0x51)
Accepts: DOCK_REQUEST (0x01), HEARTBEAT (0x04), FILE_INDEXED (0x11), CLASSIFICATION_REQUEST (0x50)

---

KEY FILES

SOURCE: /repo/Catasorter/
INDEX: src/index.js
TOOLS: src/tools/index.js, src/tools/classify-document.js
CLASSIFICATION: src/classification/glec-classifier.js
NAMING: src/naming/dewey-generator.js, src/naming/filename-builder.js
PK-INTEGRATION: src/pk-integration/nugget-extractor.js
INTERLOCK: src/interlock/socket.js, src/interlock/tumbler.js
HTTP: src/http/server.js
WEBSOCKET: src/websocket/server.js
DATABASE: src/database/client.js
CONFIG: config/server.json, config/interlock.json

DEPENDENCIES: @modelcontextprotocol/sdk, better-sqlite3, zod, dotenv, express, ws
