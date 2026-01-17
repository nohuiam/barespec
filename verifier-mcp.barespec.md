SERVER: verifier-mcp
VERSION: 1.2
UPDATED: 2026-01-16
STATUS: Production
PORT: 3021 (UDP/InterLock), 8021 (HTTP), 9021 (WebSocket)
MCP: stdio transport (stdin/stdout JSON-RPC)
DEPS: Anthropic API (Pass A/B evaluation), HuggingFace (RoBERTa NLI)
PURPOSE: Adversarial fact-checking with dual-polarity claim verification
CONFIG: /repo/verifier-mcp/config/interlock.json

---

ARCHITECTURE

WORKFLOW: 4-layer architecture (MCP stdio, InterLock UDP, HTTP REST, WebSocket)
PRINCIPLE: The unit of verification is a CLAIM, not a document
PIPELINE: Claimify extraction → V5 Fast-path (NLI → Stage2-NLI → Pass B)
SCORING: CorXFact dual-polarity (support + contradiction)
INTERLOCK: BaNano protocol, signals 0xE0-0xE3 (VERIFY_REQUEST, VERIFY_RESPONSE, CLAIM_RESOLVED, BATCH_COMPLETE)

---

TOOLS (3)

TOOL: extract_claims
INPUT: { text: string (required), context?: string, include_implicit?: boolean (default: true) }
OUTPUT: { claims: [{ id, text, type, scope, confidence }], source_text }
USE: Surface all factual assertions (explicit + implicit) from text using Claimify pipeline
EXAMPLE: extract_claims({ text: "HTTP 200 means success." })
NOTES: Surfaces explicit + implicit assertions. Types: explicit (stated), implicit (assumed), derived (inferred), normative_as_fact (opinion as fact).

TOOL: verify_claims
INPUT: { domain: string (required), claims: [{ id, text, confidence?, depends_on? }] (required), context?: { use_case?, audience?, claim_date? }, options?: { strictness?, max_sources?, enable_adversarial?, enable_fast_path? } }
OUTPUT: { summary: { verified, challenged, rejected, ambiguous }, results: [{ id, status, confidence, verdict, supporting_sources, qualifiers, counterexamples }], source_credibilities }
USE: Dual-polarity adversarial verification with Pass A (support) and Pass B (adversarial)
EXAMPLE: verify_claims({ domain: "software", claims: [{ id: "c1", text: "HTTP 200 indicates success" }] })
NOTES: V5 fast-path: Stage1-NLI → Stage2-NLI → Pass B. enable_fast_path=true handles ~80% of claims locally (free). Stage 2 uses RoBERTa trained on FEVER for fact-checking. Dependency propagation: if depends_on claim fails, dependent auto-downgrades.

TOOL: list_sources
INPUT: { domain?: string, query?: string, include_superseded?: boolean (default: false) }
OUTPUT: { sources: [{ id, title, domain, authority, type, published, trust_level }], stats: { total, byDomain, byType } }
USE: Query available authoritative sources in the registry
EXAMPLE: list_sources({ domain: "software" })
NOTES: Sources have trust_level 0-1. Superseded sources replaced by newer versions. Use before verify_claims to understand available evidence.

---

VERIFICATION FLOW

V5 FAST-PATH:
1. Stage 1 NLI: Local DeBERTa-v3 (transformers.js), entailment/contradiction/neutral
2. Stage 2 NLI: HuggingFace API (ynie/roberta-large-snli_mnli_fever_anli), contradiction detection
3. Early Exit: If high confidence (>0.85), skip Pass B
4. Pass B: LLM adversarial evaluation (constraints, edge cases)

MODELS:
- Stage 1: Xenova/deberta-v3-base-mnli-fever-anli (local ONNX, FREE)
- Stage 2: ynie/roberta-large-snli_mnli_fever_anli_R1_R2_R3-nli (HF API, trained on FEVER)

VERDICTS: SUPPORTED, SUPPORTED_WITH_QUALIFIERS, CONTRADICTED, AMBIGUOUS, UNSUPPORTED

DOMAINS:
- software: RFCs, API specs, language docs
- legal: Regulations, court decisions
- technical: Standards, specifications

STRICTNESS LEVELS:
- low: Accept weak evidence, fewer passes
- medium: Balanced verification (default)
- high: Require strong multi-source agreement

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
GET  /api/tools                  → List all MCP tools (bop-gateway)
POST /api/tools/:toolName        → Execute MCP tool via HTTP (bop-gateway)

---

WEBSOCKET EVENTS

S→C connected                    → Connection established
S→C verification_started         → Batch verification begun
S→C claim_progress               → Per-claim progress update
S→C verification_complete        → All claims processed
C→S subscribe                    → Subscribe to event types
C↔S ping/pong                    → Keepalive

---

TELEMETRY FIELDS

- nli_entailment, nli_neutral, nli_contradiction: Stage 1 NLI scores
- stage2_contradiction_score: Stage 2 RoBERTa contradiction score
- stage_exit: "NLI"|"MINICHECK"|"PASS_B": Where fast-path exited
- overconfidence_gap: Difference between claimed and actual confidence
- cost_usd: API cost for this claim verification

---

LAYERS

1. MCP stdio (3 tools) - Primary interface
2. InterLock UDP mesh (port 3021) - Peer communication
3. HTTP REST API (port 8021) - External integrations
4. WebSocket real-time (port 9021) - Live updates

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
