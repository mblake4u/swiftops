# ADR-007: SwiftOps Project Closeout

**Date:** 2026-05-19
**Status:** Accepted
**Deciders:** Michael Blake

---

## Context

SwiftOps started in March 2026 with two parallel 50/50 framings that were held live throughout the project:

1. **Product ↔ Portfolio.** Build something genuinely useful *and* a credible technical portfolio for Solutions Architect / API Integration / AI Tooling roles. Neither dominates.
2. **Commercial ↔ Open-source.** The Track 3 commercialisation direction (Fork 3 — an MCP integration layer over BYO data licences) was specifically designed to remain reversible: the architecture (BYO data, multi-source, MCP-native) worked under either end-state. The OSS-vs-commercial decision was deferred until first design-partner conversations would close.

Two prior project-level pivots are documented:

- **2026-04-23 — Pivot 1 (Zero-Footprint Approach).** Moved away from the Swift Microgateway lab framing toward the direct-sandbox Docker stack that became Track 2.
- **2026-04-30 — Pivot 2 (Path & Fork Direction).** Selected Fork 3 over Forks 1 and 2 based on the structured deep-research pre-flight. Captured in ADRs 004 / 005 / 006 (private, Notion-only — sensitive design-partner content).

By mid-May 2026 the Fork 3 work had clarified a deeper finding: the **agentic payments infrastructure category** is genuinely empty across multiple rails — Stripe MCP, Coinbase x402, Marqeta MCP, Alipay MCP, PayPal Agentic Toolkit all sit in transactional Bucket C but on card / ACH / stablecoin rails. No neutral aggregator, no published evaluation methodology, no agent-identity story. That category is materially bigger than what Fork 3 could be — and **greenfield commercial** rather than 50/50 ambiguous. A separate project sitting in agent-to-agent commerce is the better commercial vehicle.

This ADR records the decision to close SwiftOps out cleanly and redirect commercial energy to the successor project.

---

## Decision

**Close SwiftOps out as an open-source portfolio piece.** Both 50/50 framings collapse:

- **Product ↔ Portfolio → portfolio-only.** Track 2's working stack is the deliverable; methodology rigour (the published MCP-gateway evaluation methodology) is the highest-leverage artefact. No further commercial pursuit on SwiftOps.
- **Commercial ↔ Open-source → open-source-only.** All five repos remain public and stay public. No private fork. No paid licensing.

**Redirect commercial energy** to a separate, deliberately-scoped successor project (kept under wraps until first design-partner conversations close). The successor project's architecture and methodology pillars draw on explicit, named carry-forwards from SwiftOps — not blind context inheritance.

Specific carry-forwards (deliberate migration only):

- Regulatory findings — PERG 15 Annex 3(j) tech-vendor carve-out analysis; the three-bucket Path 4 cliff (informational / recommendation-with-approval / autonomous-transactional); MTL/EMI/PI line-crossing patterns.
- Agent-identity permissioning candidates — OAuth 2.1 + PKCE / OAuth + RFC 9396 RAR / DPoP / Dynamic Client Registration + mTLS / SPIFFE-SPIRE.
- Evaluation methodology — direct seed for the successor's evals-as-marketing pillar.
- Working pattern — ADRs for decisions, smoke tests per service, evidence over opinion, methodology rigour as portfolio signal.

Everything else stays in SwiftOps.

---

## Options Considered

| Option | Notes |
|---|---|
| **Close SwiftOps out as portfolio + OSS; redirect commercial energy (this ADR)** ✅ | Resolves both 50/50 framings cleanly. Preserves the methodology artefact and the deep-research record. Sets up the successor project to inherit only what's explicitly named. |
| Continue both SwiftOps Fork 3 and the new successor project in parallel | Solo-founder time budget makes this infeasible. Half-building both is worse than fully building one. |
| Continue Fork 3 only, defer the successor project | Reverses the priority signal. The successor sits in a materially larger category; deferring it spends limited time on the smaller opportunity. |
| Abandon SwiftOps entirely (delete repos, take down Substack) | Loses the open-source portfolio artefact and the methodology piece. The work is genuinely useful; closing-out-cleanly is strictly better than abandoning. |
| Keep SwiftOps private, fold into the successor project | Loses the portfolio signal that motivated the project's twin-goal framing from day one. The methodology artefact's recruiter-facing value depends on being publicly accessible. |

---

## Consequences

- **All five SwiftOps repos stay public and continue to work.** No code is removed. Latest commits remain the canonical state. Repos will be marked archived (read-only) per B-list step 11.
- **No further commercial pursuit on SwiftOps.** The Fork 3 anchor-customer outreach is not pursued (the Notion anchor dossier remains private and archived; pre-contact state preserved).
- **Methodology rigour pattern travels** as the project's most reusable contribution — both as a portfolio signal and as a direct architectural seed for the successor project's evaluation pillar.
- **Twin 50/50 framings did real work.** Holding both axes live until evidence demanded resolution meant design choices stayed reversible. Closing out without breaking prior commitments was only possible because no prior commitment had pinned an end-state.
- **ADRs 004 / 005 / 006 (private)** remain authoritative for the closed commercial paths (Path 2c structurally blocked, Forks 1/2 closed, Fork 3 selected, design partner identified). They are kept in Notion (private) and not migrated to the public repo. This ADR is the public-facing closeout document; the prior three are the private commercial record.
- **Two pivots and a closeout** is now the project's narrative shape. Captured publicly in [`docs/DEVLOG.md`](DEVLOG.md) and on the [Substack closeout post](https://mblake4u.substack.com/p/mcp-for-swift-messaging-api-two-pivots).
- **Successor project remains under wraps.** Not named, not linked, no public artefacts referencing it directly. Will become public on its own schedule once the design-partner posture is right.

---

## References

- [`docs/DEVLOG.md`](DEVLOG.md) — project narrative
- [Substack closeout post](https://mblake4u.substack.com/p/mcp-for-swift-messaging-api-two-pivots)
- [Evaluating an MCP Gateway: A Methodology](https://www.notion.so/Evaluating-an-MCP-Gateway-A-Methodology-364aa5f988dd8038bb21d34880ca6eab) (Notion)
- ADRs 004 / 005 / 006 in Notion ADR Register (private — commercial-decision record)
- swift-mcp-gateway ADR-003 (evaluation methodology — direct portfolio seed for the successor project)
