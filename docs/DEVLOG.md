# SwiftOps — Project Devlog

**Period:** March 2026 – May 2026
**Status (2026-05-18):** Closed out as a portfolio + open-source artefact. Predecessor to a separate commercial project in early scoping (not yet public).

---

## TL;DR

SwiftOps was a personal R&D project building Swift Alliance Cloud API tooling, structured around twin goals — useful product *and* a credible technical portfolio for Solutions Architect / API Integration / AI Tooling roles. Over three months it went through two pivots: a technical one (away from the Swift Microgateway, toward a direct-sandbox zero-footprint Docker stack) and a strategic one (an evidence-based decision to commercialise via Fork 3 — an MCP integration layer over BYO data licences — anchored on a small regional bank as design partner).

The strategic pivot then triggered a third move: a successor commercial project in agent-to-agent commerce, better matched to the actual opportunity that surfaced. SwiftOps closed out in May 2026 with a published evaluation methodology for the MCP gateway, a five-cluster failure analysis, a probe-found bug fix, and everything wrapped up as an open-source portfolio piece.

This devlog is the project's final canonical artefact. The README, ADRs, Notion pages, and repo history are all still here — this file is the index and narrative that ties them together.

---

## Origins — March 2026

The starting question was practical: *if I want to demonstrate Solutions Architect / API Integration / AI Tooling skills in 2026, what do I build?* Swift API tooling was an answer that sat at three intersections: (1) my banking background, (2) the under-served Swift Developer Portal ecosystem, and (3) the just-emerging Model Context Protocol space.

The first design was a Swift Microgateway (MGW) test lab on Windows + Docker, with a Python MCP server driving it via the MGW management APIs. By 2026-03-31 it was running end-to-end: Claude Desktop → MCP server → MGW Docker → sandbox.swift.com. A real OAuth flow, a real API call, a real Swift sandbox response. Three-week stretch from zero to working stack.

Documented in [Project Vision & Goals](https://www.notion.so/34baa5f988dd814d94f4e16190ccecc5) (Notion).

---

## Pivot 1 — Zero-Footprint Approach (2026-04-23)

After a few weeks of MGW work, the framing started feeling wrong for the portfolio goal. The Microgateway is a real piece of infrastructure with operational depth — but the depth is on operating the gateway, not on building agent-facing tooling. For the audience I cared about (recruiters in AI infra / API integration / solutions architecture), the *agent-facing* surface was the more interesting story.

I pivoted to a **zero-footprint Docker stack** that talked directly to `sandbox.swift.com` over OAuth + JWT-Bearer (RFC 7523), no MGW required. The decision is documented in [Project Pivot — Zero-Footprint Approach](https://www.notion.so/34baa5f988dd814197d2dd01aa2e08c2).

What got built (Track 2):

- **`swift-token-server`** (Flask + gunicorn, port 82) — fetches OAuth Bearer tokens from `sandbox.swift.com` via the RFC 7523 JWT-Bearer grant, caches them, and exposes a `/proxy/<path>` route that injects the token + handles CORS for the browser-side UI.
- **`swift-swagger-ui`** (nginx, port 83) — Swagger UI with the Swift Messaging API v2.1.0 OpenAPI spec baked in at build time. Browser-visible API surface for exploration.
- **`swift-ui-client`** (FastAPI + HTMX, port 84) — dashboard pulling live distributions data, with a JSON API at `/api/*` for a future React migration.
- **`swift-mcp-gateway`** (FastMCP, port 85) — the agent-facing piece. Exposes 10 Swift API operations (distributions, FIN/MT messages, InterAct/MX messages) as MCP tools. Auth fully transparent: all calls route through the token server's `/proxy` so the gateway itself stays credential-free.

Throughout: ADRs on key decisions, smoke tests per service, GitHub Actions CI (pip-audit + docker build + smoke tests on every push), `LICENSE` + `CONTRIBUTING.md` on each repo. Discipline at prototype scale.

One notable side-quest: scanning the open-source landscape for prior art surfaced `swiftinc/api-sample-code`, which revealed that mutating Swift API requests (POST/PUT/PATCH/DELETE) require an `X-SWIFT-Signature` header. That had been silently missing from my proxy — fixed in v0.3.2-dev and documented as a gap caught by "search before building" discipline.

---

## Pivot 2 — Path & Fork Direction (2026-04-30)

By late April the Track 2 stack was working and the obvious next question was: *could this be commercialised, or does it stay a pure portfolio piece?* I committed to making that decision on evidence rather than gut. Specifically:

- A structured **deep research pre-flight** evaluating 5 commercialisation paths × multiple operating-entity structures (UK Ltd primary plus alternatives), against the major UK / EU / US / regional regulatory regimes.
- A **competitive landscape pass** mapping the four-tier Pack C (BIC / IBAN / SSI / LEI reference data) and Pack A (cross-border payment intelligence) markets.

What the research surfaced:

- **Path 2c (retail funds-routing) is structurally blocked.** PSRs 2017 reg.138 / PSD2 / 18 USC §1960 / equivalent local banking-law regimes — across every entity structure evaluated. Closed under [ADR-005](https://www.notion.so/34caa5f988dd81d98475cf78440828cc).
- **Fork 1 (Pack C as data redistribution) is not viable for a solo founder.** LexisNexis Risk Solutions and LSEG Risk Intelligence dominate the top of the market; iban.com-class providers set a commodity €30–€300/mo price floor at the bottom. SwiftRef Distribution Licence economics fail below a five-figure paying customer count.
- **Fork 2 (Pack A as VoP/BAV specialist) is incumbent-blocked.** iPiD and SurePay are both Swift Enabler partners with bank reference customers, riding the EU Instant Payments Regulation tailwind that came into effect 9 October 2025. The window is largely closed.
- **The empty niche: the MCP-as-distribution layer for cross-bank reference and tracking data.** Adjacent finance MCPs exist (Grasshopper/Narmi, Griffin, Stripe, Slash) but none addresses multi-source ref/tracking aggregation as of mid-2026.

The decision: **Fork 3 — an MCP integration layer over BYO data licences**, multi-source, customer brings their own SwiftRef / gpi / CoP entitlements. Sells to banks (slow, sticky, high-ACV, via Path 1) AND corporate treasury / accounts payable (mid-cycle, direct). Operating entity: UK Ltd primary, Netlink-sponsored for Swift connectivity. Anchor design partner: a small regional commercial bank (identity kept private; pre-outreach as of project close).

Documented in [Project Pivot 2 — Path & Fork Direction](https://www.notion.so/352aa5f988dd8130a53bc27473c56b46), [Commercialisation Strategy](https://www.notion.so/352aa5f988dd818aa719c74e1cf14885), and [Fork 3 — MCP Integration Layer Concept](https://www.notion.so/352aa5f988dd814ca819f0e89e68fe52). [ADRs 004 / 005 / 006](https://www.notion.so/34caa5f988dd81d98475cf78440828cc) raised.

Crucially, the 50/50 product↔portfolio and 50/50 commercial↔open-source framings were preserved. The Fork 3 architecture (BYO data, multi-source, MCP-native) works for both end states; the decision was deferred until first design-partner conversations would close.

---

## Pivot 3 — Closeout in favour of a successor project (May 2026)

The Fork 3 work clarified something deeper: the **agentic payments infrastructure** category was genuinely empty, not just at the Swift correspondent layer. Stripe MCP, Coinbase x402, Marqeta MCP, Alipay MCP, PayPal Agentic Toolkit all sit in transactional Bucket C but on card / ACH / stablecoin rails. No neutral aggregator. No published evaluation methodology. No agent-identity story.

That category is bigger than what Fork 3 could be — and importantly, it's **greenfield commercial** rather than the 50/50 product↔portfolio ambiguity that SwiftOps was structured around. The right move was to start a new project — a non-custodial, rail-neutral angle in agent-to-agent commerce — with commercial intent as a single goal, and let SwiftOps close out as the portfolio + methodology predecessor.

The decision was made the second week of May. The successor project is structured with explicit carry-forward of specific learnings only, rather than blind context inheritance — a small discipline that keeps the new framing clean. A few of those carries-forward:

- **Regulatory findings** — PERG 15 Annex 3(j) tech-vendor carve-out analysis, the three-bucket Path 4 cliff (informational / recommendation-with-approval / autonomous-transactional), MTL/EMI/PI line-crossing patterns.
- **Agent-identity permissioning candidates** — OAuth 2.1 + PKCE / OAuth + RFC 9396 RAR / DPoP / Dynamic Client Registration + mTLS / SPIFFE-SPIRE. OAuth 2.0 alone was designed for human-delegated access; agents need more.
- **Evaluation methodology** — direct seed for the successor's evals-as-marketing pillar.
- **Working pattern** — ADRs for decisions, smoke tests per service, evidence over opinion, methodology rigour as a portfolio signal.

Everything else stays in SwiftOps.

---

## Final session — A4a evaluation methodology

The closeout's centrepiece work was shipping a **published evaluation methodology** for the MCP gateway. This was the highest-leverage portfolio artefact to close out on: methodology rigour is the rarest engineering trait in the MCP-gateway space as of mid-2026 (every adjacent gateway ships tools and docs; none ship a published eval framework). It also doubles as a direct seed for the successor project's evals-as-marketing pillar.

The methodology has **three orthogonal axes**:

1. **Tool-selection accuracy** — given a natural-language question, does the model pick the correct MCP tool? Strict-equality scored.
2. **Argument fidelity** — given the right tool, are the right arguments populated? Strict equality for typed args; LLM-as-judge with explicit rubric for free-text args (payloads, reasons).
3. **Operational metrics** — latency (P50, P95 per tool, live sandbox, 20 runs each) and coverage (% of upstream API endpoints reachable). Mutating tools deliberately not probed.

30 hand-curated test cases (18 selection / 6 argument / 6 negative). Multi-model harness using the Anthropic API directly with prompt caching on system + tools (96.7% cache hit ratio in practice — ~10× cost reduction). LLM-judge model pinned for stability across runs.

Two runs published as the first baseline (2026-05-18):

- **Run A — latest each tier:** opus-4-7 (87%), sonnet-4-6 (77%), haiku-4-5 (77%). Mixed-generation, real-world relevant.
- **Run B — matched generation 4.5, fully pinned:** flat 80% across all three tiers. Methodology control.

But the headline finding wasn't the numbers. It was the **five-cluster failure analysis**:

| Cluster | Cause | Verdict |
|---|---|---|
| 1 — partial send-tool specs | Test-set authorship bug | Test set is wrong; cases need rewriting in v2 |
| 2 — singular/plural confusion | Product naming smell | Recommend `list_*` / `get_*` rename; v2 backlog |
| 3 — small-model jargon limit | Model tier capability | Real finding — Haiku failed MX→InterAct mapping; Sonnet/Opus handled it |
| 4 — refusal-instruction non-compliance | Sonnet-tier behaviour | Sonnet substitutes near-matches; Opus & Haiku follow refusal instruction |
| 5 — tool/API contract mismatch | **Found by axis 3 only** | `download_*_messages` declared optional-arg but Swift API requires it — **fixed in session** as v0.1.1 |

Cluster 5 was the most interesting outcome. The axis-3 latency probe surfaced a real production-readiness bug that neither model-evaluation axis caught — proof the axes are genuinely orthogonal. Fixed and re-deployed in the same session.

Full write-up on [Notion](https://www.notion.so/Evaluating-an-MCP-Gateway-A-Methodology-364aa5f988dd8038bb21d34880ca6eab); ADR-003 + analysis doc + raw results in the [`swift-mcp-gateway/evals/`](https://github.com/mblake4u/swift-mcp-gateway/tree/staging/evals) directory.

---

## What's open / what's closed

**Closed and deployed:**
- A4a — eval methodology, harness, baseline, analysis, probe-found fix as v0.1.1
- Track 1 — MGW lab (paused stable; foundation for any future MGW work)
- Track 2 build-out — token server / Swagger UI / UI client / MCP gateway (all v0.1.0+ shipped)
- Track 3 — direction set (Fork 3, UK Ltd + Netlink); commercial pursuit redirected to the successor project

**On the closeout backlog (may or may not ship):**
- A4b — auth/scoped-token demo (agent-identity permissioning candidates)
- A4c — engineering polish (typed Pydantic schemas, structured logging, retries)
- A3 — live demo URL (Fly.io / Railway)
- A5 — supply chain CI hardening (hash pinning, completing the pip-audit work)
- README polish across the four repos for cold readers
- Substack post + LinkedIn distillation (this devlog → distilled public versions)

**Explicitly deferred (v2 backlog of the methodology):**
- Test-set v2 (fix cluster-1, diversity audit)
- Temperature = 0 or N-run aggregation (eliminate observed non-determinism)
- Tool renaming product fix (`list_*` / `get_*`)
- Multi-turn case dimension
- Probe from co-located deployment (cleaner latency numbers)

---

## Reflections

A few things stand out from three months of this.

**Pivot early when evidence accumulates.** Both pivots happened within a week of evidence reaching me. The cost of changing direction is always lower than the cost of continuing on a route the evidence has invalidated. The MGW → zero-footprint pivot was a portfolio-framing call; the Fork 3 → successor pivot was a category-scope call. Both were the right move at the time they were made.

**Methodology rigour is the rarest portfolio signal.** Most public MCP gateways ship tools and docs. None publish an evaluation methodology. That asymmetry was visible from the OSS landscape and competitive scans — and once visible, the right thing to do was to build the methodology, not more tools. The five-cluster analysis demonstrates judgment in a way that a tools-count list cannot.

**Probe and eval are orthogonal.** Cluster 5 (the probe-found contract bug) was the most concrete proof. The two methodology axes that scored model behaviour both passed every argument-fidelity case at 100%, missed the bug entirely. The latency probe caught it. They're not redundant — each catches things the others can't.

**The 50/50 framings did real work.** Holding product↔portfolio and commercial↔OSS as live throughout meant design choices stayed reversible. The Fork 3 architecture works under both commercial and OSS end-states. The closeout decision didn't break any prior commitments because no prior commitments forced a single end-state.

**Closed decisions are an asset.** The ADR register and Notion's closed-decisions list mean future sessions (and future projects) don't re-litigate settled questions. Path 2c is closed. Fork 1 is closed. Path 4(c) is a long-term research bet, not a near-term build. That's all written down, with rationale. The successor project starts with a clean slate but doesn't have to redo the regulatory homework.

**Don't auto-import context.** The successor project deliberately doesn't inherit SwiftOps's context. Each carry-forward decision is explicit. This is a small bit of discipline but it keeps the new project's framing clean.

---

## What this project is and isn't

SwiftOps **is** a working, tested, end-to-end Swift API tooling stack with a published evaluation methodology, a deep-research-driven commercialisation analysis, ADRs for every key decision, and twelve-week chronology of two pivots and a closeout. Code is open source, fully reproducible.

SwiftOps **isn't** a commercial product, isn't a complete Swift API gateway (it covers core message flows; operational endpoints are out of v1 scope), isn't a methodology that subsumes all MCP-gateway evaluation problems (single-turn cases, single author's test-set intuition, sandbox-not-production latency).

The honest framing for a portfolio reader: this is what a solo founder shipped in three months while preserving optionality, and the methodology — not the tools — is the contribution that travels.

---

## What's next

A successor project — a non-custodial, rail-neutral intelligence layer in agent-to-agent commerce — sits in early scoping. Not yet public. The regulatory and methodology learnings from SwiftOps carry forward as explicit, named seeds.

If anything in this project is useful to you — the code, the methodology, the ADRs, the closed-decisions framework — please use it. The repo is and stays public.

---

*— Michael Blake (2026-05-18)*
