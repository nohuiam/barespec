SERVER: niws-production
VERSION: 1.0
UPDATED: 2026-01-10
STATUS: Production
PORTS: UDP 3035 | HTTP 8035 | WS 9035
MCP: stdio transport (stdin/stdout JSON-RPC)
INTERLOCK: Enabled (peer mesh communication, receives all signals)
DEPS: Claude API (LLM generation), niws-intake (articles), niws-analysis (bias data), tenets-server (moral evaluation)
PURPOSE: Content production pipeline - scripts, briefs, Christ-Oh-Meter moral ratings
CONFIG: ENV HTTP_PORT (default 8035), ANTHROPIC_API_KEY

---

ARCHITECTURE

FUNCTION: Third stage of NIWS pipeline - content production for "Warnings with Loring"
FLOW: Stories + Analysis -> Story Briefs -> Scripts -> Validation -> Export
STORAGE: SQLite (scripts.db, briefs.db)
DATABASES:
  - scriptDatabase: Scripts, sections, revisions, learned patterns
  - briefDatabase: Story briefs, quotes, legislation, Christ-Oh-Meter ratings
KEY_FEATURE: Full editorial workflow with QA validation and revision history

---

TOOL CATEGORIES

SCRIPT GENERATION (3 tools):
- generate_script: Generate complete "Warnings with Loring" script from story
- generate_section: Generate single script section (opening, verified_facts, outlet_analysis, critical_questions, closing)
- regenerate_section: Regenerate section with new parameters/guidance

SCRIPT RETRIEVAL (3 tools):
- get_script: Get script by ID with all sections
- list_scripts: List scripts with filters (story_id, status, limit, offset)
- get_script_section: Get specific section from script

SCRIPT EDITING (4 tools):
- update_script: Update script metadata (title, status)
- update_section: Update section content directly
- revise_section: AI-assisted revision based on feedback
- delete_script: Delete script

SCRIPT VALIDATION (3 tools):
- validate_script: Validate against editorial standards and QA rules
- check_prohibited_patterns: Check text for prohibited patterns (editorial, intent, truth claims, partisan)
- validate_niws_elements: Check for required NIWS elements (opening, closing intro, signature)

SCRIPT REVISIONS (2 tools):
- get_revisions: Get revision history for script
- revert_to_revision: Revert section to previous revision

SCRIPT EXPORT (2 tools):
- export_script: Export in format (markdown, plain_text, json)
- estimate_duration: Estimate reading duration at standard speaking rate

SCRIPT PATTERNS (3 tools):
- get_learned_patterns: Get learned editing patterns for improvement
- add_learned_pattern: Add pattern from editing feedback
- get_script_stats: Statistics about scripts and patterns

BRIEFS (8 tools):
- create_story_brief: Create brief from story cluster (quotes, legislation, optional morality rating)
- get_story_brief: Get brief by ID with quotes, legislation, ratings
- list_story_briefs: List briefs with filters (story_id, status, moral_alignment)
- update_brief_status: Update workflow status (draft, reviewed, approved, used)
- rate_christ_oh_meter: Rate action on Christ-Evil spectrum using 25 Gospel tenets
- compare_quotes: Find outlet variations in reporting same speaker's quotes
- analyze_legislation: Extract factual, non-opinionated effects of legislation
- get_brief_stats: Statistics about briefs and ratings

TOTAL: 28 tools (20 script + 8 brief)

---

SCRIPT SECTIONS

SECTION: opening
DESC: Hook and story introduction
ELEMENTS: Loring's catchphrase, story setup

SECTION: verified_facts
DESC: Cross-verified facts from multiple outlets
ELEMENTS: Numbered facts, source attributions

SECTION: outlet_analysis
DESC: How outlets framed the story differently
ELEMENTS: Left/Center/Right comparison, framing differences

SECTION: critical_questions
DESC: Questions the viewer should consider
ELEMENTS: Socratic-style prompts, missing context highlights

SECTION: closing
DESC: Call to action and signature
ELEMENTS: Viewer prompt, "This has been Warnings with Loring"

---

CHRIST-OH-METER

SCALE: -1.0 (Evil) to +1.0 (Christ)
TENETS: 25 Gospel tenets for moral evaluation
VERDICT_TYPES: christ | anti-christ | neutral | mixed
INTEGRATION: tenets-server via HTTP API
OUTPUT: spectrum_score, verdict, strongest_christ_tenets, strongest_evil_tenets, counterfeits_detected, reasoning

---

PROHIBITED PATTERNS

CATEGORY: editorial
DESC: Opinion or judgment phrases
EXAMPLES: "obviously", "clearly wrong", "everyone knows"

CATEGORY: intent
DESC: Claims about motivation
EXAMPLES: "intended to", "deliberately", "designed to harm"

CATEGORY: truth_claims
DESC: Absolute truth assertions
EXAMPLES: "the truth is", "fact is", "in reality"

CATEGORY: partisan
DESC: Political characterization
EXAMPLES: "liberal agenda", "conservative bias", "left-wing"

---

DATABASE SCHEMA

TABLE: scripts
COLUMNS: id, story_id, brief_id, title, status, content, word_count, estimated_duration_seconds, created_at, updated_at

TABLE: script_sections
COLUMNS: id, script_id, section_type, content, word_count, notes, order_index, created_at, updated_at

TABLE: script_revisions
COLUMNS: id, script_id, section_id, previous_content, new_content, revision_notes, created_at

TABLE: learned_patterns
COLUMNS: id, pattern_type, original, replacement, frequency, confidence, is_active, created_at

TABLE: briefs
COLUMNS: id, story_id, title, summary, key_facts, perspectives, status, christ_oh_meter_score, moral_alignment, created_at, updated_at

TABLE: brief_quotes
COLUMNS: id, brief_id, text, attribution, context, outlet_id, created_at

TABLE: brief_legislation
COLUMNS: id, brief_id, bill_number, title, summary, status, created_at

TABLE: christ_oh_meter_ratings
COLUMNS: id, brief_id, action, subject, affected, context, spectrum_score, verdict, reasoning, tenets_evaluation_id, created_at

---

HTTP ENDPOINTS

GET /api/health - Health check
GET /api/status - Server status with dependency health

GET /api/scripts - List scripts with filters
GET /api/scripts/:id - Get script by ID
POST /api/scripts - Generate new script
PUT /api/scripts/:id - Update script metadata/status
DELETE /api/scripts/:id - Delete script
POST /api/scripts/:id/validate - Validate script against QA rules
POST /api/scripts/:id/export - Export script in specified format

GET /api/briefs - List briefs with filters
GET /api/briefs/:id - Get brief with quotes, legislation, ratings
POST /api/briefs - Create story brief
PUT /api/briefs/:id/status - Update brief status

POST /api/christ-oh-meter/rate - Rate action on moral spectrum
GET /api/christ-oh-meter/ratings/:id - Get ratings for brief

GET /api/stats/scripts - Script statistics
GET /api/stats/briefs - Brief statistics

---

WEBSOCKET EVENTS

EVENT: script:generating - Script generation started
EVENT: script:generated - Script generation complete
EVENT: script:validated - Script validation complete
EVENT: script:exported - Script exported
EVENT: brief:created - New story brief created
EVENT: brief:rated - Christ-Oh-Meter rating added

---

INTERLOCK SIGNALS

EMIT:
- server:ready - Server operational
- server:shutdown - Server shutting down
- script:generated - New script created
- script:validated - Script passed QA
- script:approved - Script approved for delivery
- brief:created - New brief created
- brief:rated - Brief received morality rating
- christohmeter:rated - Standalone rating created

RECEIVE:
- * (receives all signals from mesh)

PEERS: niws-intake (3033), niws-analysis (3034), niws-delivery (3036), research-bus (3015), tenets-server (3027)

---

SERVICE CLIENTS

CLIENT: intakeClient
URL: http://localhost:8033
METHODS: getArticle, getArticlesForStory, getStory, listStories, getOutlet, listOutlets, health

CLIENT: analysisClient
URL: http://localhost:8034
METHODS: getArticleAnalysis, getComparativeAnalysis, analyzeArticle, analyzeStory, health

CLIENT: researchClient
URL: http://localhost:8015
METHODS: search, getCoverageAnalysis, health

CLIENT: tenetsClient
URL: http://localhost:8027
METHODS: evaluate, getTenets, health

---

KEY FILES

SOURCE: /repo/niws-production/
INDEX: src/index.ts
HTTP: src/http/server.ts
WS: src/websocket/server.ts
INTERLOCK: src/interlock/
TOOLS:
  - src/tools/scriptTools.ts (20 tools)
  - src/tools/briefTools.ts (8 tools)
DATABASE:
  - src/database/scriptDatabase.ts
  - src/database/briefDatabase.ts
SERVICES:
  - src/services/scriptGenerator.ts
  - src/services/christOhMeter.ts
  - src/services/briefExtractor.ts
  - src/services/clients.ts
CONFIG:
  - config/interlock.json
  - src/config/prohibitedPatterns.ts
  - src/config/sectionTemplates.ts
VALIDATION: src/validation/schemas.ts
DATA: data/scripts.db, data/briefs.db
