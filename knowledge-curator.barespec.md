SERVER: knowledge-curator
VERSION: 1.1
UPDATED: 2025-12-26
STATUS: Production
PORT: 3017 (UDP/InterLock), 8017 (HTTP), 9017 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
PURPOSE: Knowledge transformation with 70-90% token reduction while preserving 100% essential information
CONFIG: /repo/knowledge-curator/config/interlock.json

---

ARCHITECTURE

WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
DATABASE: SQLite (better-sqlite3)
COMPRESSION: SCOPE method (Semantic Chunking + Rewriting)
GLEC: Genesis (strategy), Leviticus (process), Exodus (implementation), Context (reference)
INTERLOCK: BaNano protocol, signals 0xB0-0xBF (CURATE_REQUEST, CURATE_COMPLETE)

---

TOOLS (6)

TOOL: curate_knowledge
INPUT: { filePath: string (required), classification: { glec: "Genesis"|"Leviticus"|"Exodus"|"Context" (required), subject: string (required), confidence: 0-1 (required), keywords: string[] (required) }, options?: { compressionLevel: "low"|"medium"|"high", preserveFormatting: boolean, generateCrossRefs: boolean } }
OUTPUT: { pk_entry: { id, compressed_content, token_count, compression_ratio }, nuggets_extracted: [], cross_refs: [] }
USE: Transform classified document into optimized PK entry
EXAMPLE: curate_knowledge({ filePath: "/path/doc.md", classification: { glec: "Genesis", subject: "Architecture", confidence: 0.9, keywords: ["design", "system"] } })

TOOL: analyze_nugget_patterns
INPUT: { nuggetIds: string[] (required), analysisType: "frequency"|"relationships"|"trends" (required) }
OUTPUT: { patterns: [], insights: [], recommendations: [] }
USE: Analyze patterns in gold nuggets for insights
EXAMPLE: analyze_nugget_patterns({ nuggetIds: ["nug_001", "nug_002"], analysisType: "relationships" })

TOOL: update_search_index
INPUT: { pkId: string (required), content: string (required), keywords: string[] (required), metadata?: object }
OUTPUT: { indexed: boolean, index_id, searchable_terms }
USE: Update search index with a PK entry
EXAMPLE: update_search_index({ pkId: "pk_001", content: "Document content", keywords: ["key1", "key2"] })

TOOL: build_cross_references
INPUT: { sourceId: string (required), targetIds: string[] (required), referenceType: "cites"|"extends"|"contradicts"|"supports" (required) }
OUTPUT: { references_created: number, graph_updated: boolean }
USE: Build typed relationships between documents
EXAMPLE: build_cross_references({ sourceId: "doc_001", targetIds: ["doc_002", "doc_003"], referenceType: "extends" })

TOOL: get_compression_stats
INPUT: { timeRange?: "hour"|"day"|"week"|"month" }
OUTPUT: { total_documents, avg_compression_ratio, tokens_saved, processing_time_avg }
USE: Get compression performance metrics
EXAMPLE: get_compression_stats({ timeRange: "week" })

TOOL: query_knowledge_graph
INPUT: { startNode: string (required), depth: number (required, max: 5), relationshipTypes?: string[] }
OUTPUT: { nodes: [], edges: [], paths: [] }
USE: Query document relationships in knowledge graph
EXAMPLE: query_knowledge_graph({ startNode: "doc_001", depth: 2, relationshipTypes: ["extends", "cites"] })

---

KEY FILES

SOURCE: /repo/knowledge-curator/
INDEX: src/index.ts
TYPES: src/types.ts
DATABASE: src/database/schema.ts
COMPRESSION: src/compression/compressor.ts
CROSSREF: src/crossref/builder.ts
