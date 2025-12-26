TOOLS: verifier-mcp
VERSION: 1.0
UPDATED: 2025-12-26
COUNT: 3

---

TOOL: extract_claims
INPUT: { text: string (required), context?: string, include_implicit?: boolean (default: true) }
OUTPUT: { claims: [{ id: string, text: string, type: "explicit"|"implicit"|"derived"|"normative_as_fact", scope: "general"|"conditional"|"domain_specific"|"temporal"|"definition", confidence: number }], source_text: string }
USE: Extract all factual claims from text using Claimify 5-stage pipeline
EXAMPLE: extract_claims({ text: "HTTP 200 means the request succeeded. REST APIs should always return JSON." })
NOTES: Surfaces explicit + implicit assertions. Types: explicit (stated), implicit (assumed), derived (inferred), normative_as_fact (opinion as fact).

---

TOOL: verify_claims
INPUT: { domain: string (required), claims: [{ id: string, text: string, confidence?: number, depends_on?: string[] }] (required), context?: { use_case?: string, audience?: string, claim_date?: string }, options?: { strictness?: "low"|"medium"|"high" (default: "medium"), max_sources?: number (default: 5), enable_adversarial?: boolean (default: true), enable_fast_path?: boolean (default: true) } }
OUTPUT: { summary: { verified: number, challenged: number, rejected: number, ambiguous: number }, results: [{ id: string, status: "SUPPORTED"|"SUPPORTED_WITH_QUALIFIERS"|"CONTRADICTED"|"AMBIGUOUS"|"UNSUPPORTED", confidence: number, confidence_interval: [number, number], verdict: string, supporting_sources: [{ source_id, section, quote, correlation_strength }], qualifiers: string[], counterexamples: string[], failure_modes: string[], edge_cases: string[], recommendation?: string, grade: "A"|"B"|"C"|"D"|"F", telemetry?: object }], source_credibilities: Record<string, number> }
USE: Dual-polarity adversarial verification with Pass A (support) and Pass B (adversarial)
EXAMPLE: verify_claims({ domain: "software", claims: [{ id: "c1", text: "HTTP 200 indicates success" }], options: { strictness: "high" } })
NOTES: V5 fast-path: NLI → MiniCheck → Pass B. enable_fast_path=true handles ~80% of claims locally (free). Dependency propagation: if depends_on claim fails, dependent auto-downgrades.

---

TOOL: list_sources
INPUT: { domain?: string, query?: string, include_superseded?: boolean (default: false) }
OUTPUT: { sources: [{ id: string, title: string, domain: string, authority: string, type: string, published: string, trust_level: number, superseded_by?: string, effective_from?: string, effective_until?: string|null }], stats: { total: number, byDomain: Record<string, number>, byType: Record<string, number> } }
USE: Query available authoritative sources in the registry
EXAMPLE: list_sources({ domain: "software", query: "HTTP" })
NOTES: Sources have trust_level 0-1. Superseded sources replaced by newer versions. Use before verify_claims to understand available evidence.

---

DOMAINS:
- software: RFCs, API specs, language docs
- legal: Regulations, court decisions
- technical: Standards, specifications

STRICTNESS LEVELS:
- low: Accept weak evidence, fewer passes
- medium: Balanced verification (default)
- high: Require strong multi-source agreement

VERDICTS:
- SUPPORTED: Verified with high confidence
- SUPPORTED_WITH_QUALIFIERS: Verified but with caveats
- CONTRADICTED: Evidence contradicts claim
- AMBIGUOUS: Mixed or conflicting evidence
- UNSUPPORTED: No evidence available

V5 FAST-PATH PIPELINE:
1. NLI (free, local): entailment/contradiction/neutral
2. MiniCheck (cheap, HuggingFace): fact-grounded consistency
3. Early exit if confidence > 0.85
4. Pass B (LLM): Full adversarial evaluation

TELEMETRY FIELDS:
- nli_entailment, nli_neutral, nli_contradiction: NLI scores
- minicheck_score: MiniCheck grounding score
- stage_exit: "NLI"|"MINICHECK"|"FULL": Where fast-path exited
- overconfidence_gap: Difference between claimed and actual confidence
- cost_usd: API cost for this claim verification
