SERVER: verifier-mcp
VERSION: 1.0
UPDATED: 2025-12-26
STATUS: Production
PORT: 3021 (UDP/InterLock), 8021 (HTTP), 9021 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: Anthropic API (Pass A/B evaluation), HuggingFace (MiniCheck NLI)
PURPOSE: Adversarial fact-checking with dual-polarity claim verification
CONFIG: /repo/verifier-mcp/config/interlock.json

---

ARCHITECTURE

WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
PRINCIPLE: The unit of verification is a CLAIM, not a document
PIPELINE: Claimify extraction → V5 Fast-path (NLI → MiniCheck → Pass B)
SCORING: CorXFact dual-polarity (support + contradiction)
INTERLOCK: BaNano protocol, signals 0xE0-0xE3 (VERIFY_REQUEST, VERIFY_RESPONSE, CLAIM_RESOLVED, BATCH_COMPLETE)

---

TOOLS (3)

TOOL: extract_claims
INPUT: { text: string (required), context?: string, include_implicit?: boolean (default: true) }
OUTPUT: { claims: [{ id, text, type, scope, confidence }], source_text }
USE: Surface all factual assertions (explicit + implicit) from text using Claimify pipeline
EXAMPLE: extract_claims({ text: "HTTP 200 means success." })

TOOL: verify_claims
INPUT: { domain: string (required), claims: [{ id, text, confidence?, depends_on? }] (required), context?: { use_case?, audience?, claim_date? }, options?: { strictness?, max_sources?, enable_adversarial?, enable_fast_path? } }
OUTPUT: { summary: { verified, challenged, rejected, ambiguous }, results: [{ id, status, confidence, verdict, supporting_sources, qualifiers, counterexamples }], source_credibilities }
USE: Dual-polarity adversarial verification with Pass A (support) and Pass B (adversarial)
EXAMPLE: verify_claims({ domain: "software", claims: [{ id: "c1", text: "HTTP 200 indicates success" }] })

TOOL: list_sources
INPUT: { domain?: string, query?: string, include_superseded?: boolean (default: false) }
OUTPUT: { sources: [{ id, title, domain, authority, type, published, trust_level }], stats: { total, byDomain, byType } }
USE: Query available authoritative sources in the registry
EXAMPLE: list_sources({ domain: "software" })

---

VERIFICATION FLOW

V5 FAST-PATH:
1. NLI Check: Local model, entailment/contradiction/neutral
2. MiniCheck: HuggingFace API, fact-grounded consistency
3. Early Exit: If high confidence (>0.85), skip Pass B
4. Pass B: LLM adversarial evaluation (constraints, edge cases)

VERDICTS: SUPPORTED, SUPPORTED_WITH_QUALIFIERS, CONTRADICTED, AMBIGUOUS, UNSUPPORTED

CLAIM TYPES:
- explicit: Direct statements
- implicit: Unstated assumptions
- derived: Logical consequences
- normative_as_fact: Opinions stated as facts

---

HTTP REST API

GET  /health                     → { status, uptime, version }
GET  /api/v1/stats               → { claims_processed, fast_path_exits, by_verdict }
POST /api/v1/claims/extract      → extract_claims via HTTP
POST /api/v1/claims/verify       → verify_claims via HTTP
GET  /api/v1/sources             → list_sources via HTTP
GET  /api/v1/sources/:domain     → list_sources filtered by domain

---

WEBSOCKET EVENTS

S→C connected                    → Connection established
S→C verification_started         → Batch verification begun
S→C claim_progress               → Per-claim progress update
S→C verification_complete        → All claims processed
C→S subscribe                    → Subscribe to event types
C↔S ping/pong                    → Keepalive

---

KEY FILES

SOURCE: /repo/verifier-mcp/
INDEX: src/index.ts
TOOLS: src/tools/
CORE: src/core/claim-extractor.ts, fast-path.ts, nli-checker.ts, minicheck.ts, pass-evaluator.ts
HTTP: src/http/server.ts
WS: src/websocket/server.ts
INTERLOCK: src/interlock/

DEPENDENCIES: @modelcontextprotocol/sdk, express, ws, @huggingface/inference, zod
