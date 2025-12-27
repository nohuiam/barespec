LIBRARY: Skill Trigger Phrases
VERSION: 1.0
UPDATED: 2025-12-27
COUNT: 13 skills
PURPOSE: Natural language triggers that activate skill workflows

---

## QUICK REFERENCE

| Skill | Primary Trigger | Tools Choreographed |
|-------|-----------------|---------------------|
| batch-strategy | "Process the Dropository" | 4 |
| build-with-llms | "Build with ChatGPT" | 6 |
| classification-workflow | "GLEC classify" | 5 |
| comet-browser | "Browse to..." | 24 CDP |
| document-evolution | "Evolve this document" | 3 |
| gnc-candidate-detection | "Is this a GNC?" | 2 |
| quality-review | "Review classifications" | 3 |
| redundancy-resolution | "Resolve duplicates" | 4 |
| research-workflow | "Research before building" | 5 |
| server-development-workflow | "Build new server" | 8+ |
| strategic-glec-classification | "Classify this document" | 0 |
| testing-protocol | "Test the server" | 6+ |
| verification-workflow | "Verify this" | 4 |

---

## BATCH-STRATEGY

**Purpose:** Analyze Dropository and execute batch processing strategy

TRIGGERS:
- "Process the Dropository"
- "Organize files in Dropository"
- "Run Catasorter on new documents"
- "Batch process inbox"
- "What's in the Dropository?"

TOOLS: filesystem-test:list_directory, filesystem-test:read_text_file, context-guardian:session_state, catasorter:batch_process_dropository

---

## BUILD-WITH-LLMS

**Purpose:** Iterative build protocol with ChatGPT and Perplexity

TRIGGERS:
- "Build with ChatGPT"
- "Report to ChatGPT"
- "Validate with Perplexity"
- "V2/V3/V4 implementation"
- "Iterative build"
- "Get ChatGPT feedback"
- "Deep research this"

TOOLS: CDP (Comet), ChatGPT via CDP, Perplexity via CDP, EnterPlanMode, TodoWrite, testing tools

---

## CLASSIFICATION-WORKFLOW

**Purpose:** Complete GLEC classification pipeline orchestration

TRIGGERS:
- "GLEC classify"
- "GLEC classify all inbox files"
- "Process the Dropository"
- "Run Catasorter"
- "Classify and organize [path]"
- "What category is [file]?"
- "Is this Genesis or Exodus?"
- "Create a PK entry for this"

TOOLS: context-guardian, smart-file-organizer, glec-classifier, quartermaster, filesystem-test

---

## COMET-BROWSER

**Purpose:** Browse web and automate websites via CDP

TRIGGERS:
- "Browse to [url]"
- "Search Perplexity for..."
- "Open [website]"
- "Automate this website"
- "Take a screenshot of..."
- "Fill out this form"
- "Click on..."
- "Get the text from..."

TOOLS: 24 CDP commands (navigate, getText, click, type, screenshot, etc.)

---

## DOCUMENT-EVOLUTION

**Purpose:** Evolve documents to complete barespec format

TRIGGERS:
- "Evolve this document"
- "Upgrade to barespec"
- "Complete this specification"
- "Expand this documentation"
- "Convert to barespec format"

TOOLS: Read, Write, filesystem operations

---

## GNC-CANDIDATE-DETECTION

**Purpose:** Detect Gold Nugget Card candidates vs preserve-as-is documents

TRIGGERS:
- "Is this a GNC?"
- "Should I extract this?"
- "Is this extractable?"
- "Preserve or extract?"
- "Check document type"
- "Is this creative content?"

TOOLS: filesystem-test:read_text_file, catasorter integration

---

## QUALITY-REVIEW

**Purpose:** Review and validate batch classifications

TRIGGERS:
- "Review classifications"
- "Check low-confidence results"
- "Validate these classifications"
- "Review ambiguous files"
- "Quality check the batch"
- "Show uncertain classifications"

TOOLS: filesystem-test:read_text_file, strategic-glec-classification skill, batch results

---

## REDUNDANCY-RESOLUTION

**Purpose:** Resolve duplicate and similar documents

TRIGGERS:
- "Resolve duplicates"
- "Merge similar files"
- "Handle redundant documents"
- "Clean up duplicates"
- "Deduplicate these files"
- "Compare these documents"

TOOLS: filesystem-test:read_text_file, smart-merger:smart_merge, safe-batch-processor:process_batch, filesystem-test:write_file

---

## RESEARCH-WORKFLOW

**Purpose:** Exhaustive research before building

TRIGGERS:
- "Research before building"
- "Deep research [topic]"
- "What do I need to know about..."
- "Investigate [technology]"
- "Research phase for [project]"
- "Fill knowledge gaps"

TOOLS: research-bus:perplexity_search, research-bus:deep_research, research-bus:citation_vault, looker:enhanced_search, enterspect:spatial_search

---

## SERVER-DEVELOPMENT-WORKFLOW

**Purpose:** Build MCP servers using 4-layer architecture

TRIGGERS:
- "Build new server"
- "Create MCP server for..."
- "Implement [server-name]"
- "Start server development"
- "Build the [feature] server"

TOOLS: MCP SDK, InterLock, HTTP server, WebSocket, database, Context Guardian, Tumbler, testing

---

## STRATEGIC-GLEC-CLASSIFICATION

**Purpose:** Comprehensive GLEC analysis with percentages

TRIGGERS:
- "Classify this document"
- "What's the GLEC breakdown?"
- "Analyze document category"
- "Rate this document"
- "GLEC analysis"
- "What type of document is this?"

TOOLS: None (pure analysis skill)

---

## TESTING-PROTOCOL

**Purpose:** Validate server builds before production

TRIGGERS:
- "Test the server"
- "Run test suite"
- "Validate the build"
- "Check coverage"
- "Stress test"
- "Integration tests"
- "Is it production ready?"

TOOLS: Vitest, database, InterLock mesh, stress test tools, Claude Desktop

---

## VERIFICATION-WORKFLOW

**Purpose:** Fact-check documents using verifier-mcp

TRIGGERS:
- "Verify this document"
- "Fact-check this"
- "Is this accurate?"
- "Check the claims in..."
- "Verify before publishing"
- "Reality-check this content"
- "Are these facts correct?"

TOOLS: filesystem-test:read_text_file, verifier-mcp:extract_claims, verifier-mcp:list_sources, verifier-mcp:verify_claims

---

## TRIGGER PATTERNS

### Immediate Execution (No Questions)
- "GLEC classify" → classification-workflow
- "Verify this" → verification-workflow
- "Process the Dropository" → batch-strategy

### Requires Context
- "Classify this" → Needs file path or content
- "Build new server" → Needs specification
- "Deep research" → Needs topic

### Chain Triggers
- "Process and verify" → batch-strategy → verification-workflow
- "Classify then review" → classification-workflow → quality-review
- "Research then build" → research-workflow → server-development-workflow

---

## USAGE NOTES

1. Triggers are case-insensitive
2. Partial matches work ("GLEC" matches "GLEC classify")
3. Context from conversation is used to disambiguate
4. Multiple skills can chain in sequence
5. User override always takes precedence
