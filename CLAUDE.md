# SwiftOps — Claude Code Context

---

## Project Overview

SwiftOps is a personal R&D project building Swift Alliance Cloud API tooling on a zero-footprint Docker stack, with an active commercialisation direction (Track 3 / Fork 3) under exploration.

**Two parallel goals, kept in 50/50 balance — do NOT collapse to one:**

1. **Product** — build genuinely useful Swift API tooling, MCPs, and services
2. **Portfolio** — demonstrate skills for Solutions Architect / API+Integration Specialist / AI Tooling Engineer roles

**Open-source remains a viable end-state for Track 3.** The OSS-vs-commercial decision is deferred until first design-partner conversations (SCB) close. Never assume commercial-only.

---

## Three Tracks

Following two pivots (2026-04-23 zero-footprint, 2026-04-30 path/fork direction), SwiftOps operates three tracks in parallel. Don't mix track contexts.

| Track | Purpose | Status |
|---|---|---|
| **Track 1 — MGW lab** | Production-grade Swift Microgateway test lab. Foundation for any future MGW-based work. | Stable, paused |
| **Track 2 — Zero-Footprint Docker stack** | Direct sandbox API dev infrastructure. Token server + Swagger UI + UI client + planned MCP gateway. | Active development |
| **Track 3 — Fork 3 Commercialisation Direction** | Product direction crystallised: MCP integration layer over BYO data licences, anchored on SCB. Commercial vs open-source end-state undecided. | Active scoping |

The **Swift API MCP gateway is the convergence point of Track 2 (build) and Track 3 (commercialise)**.

When the user asks about "roadmap", "status", or "what's next", first clarify which track — or answer per-track.

---

## Track 3 — Product Direction (Fork 3)

**Fork 3:** an **MCP integration layer over BYO (bring-your-own) data licences**. Multi-source. Sells to banks (Path 1 channel — slow, sticky, high-ACV) AND corporate treasury / accounts payable (direct, mid-cycle).

The customer brings their SwiftRef licence, gpi entitlement, CoP/VoP scheme membership. Fork 3 provides the agent surface, multi-source resolution, auth/tenancy model, and published evals — not the underlying data.

**Decided (do not reopen):**
- **Paths in scope:** Path 1 (bank-internal tooling) + Path 4 read-only Buckets A (informational) and B (recommendation, human approves)
- **Anchor customer:** SCB = **Southern Commercial Bank N.V.** (SCOM Bank), Tourtonnelaan 33, Paramaribo. ~26 employees. Website: `scombank.sr`. **NOT** De Surinaamsche Bank N.V. (DSB) — DSB is on Fiserv and is sometimes mis-attributed; do not confuse. Hard rule (ADR-006).
- **Operating entity:** UK Ltd primary
- **Execution route:** Netlink-sponsored (Netlink holds Alliance Cloud / Alliance Connect Virtual / Swift Business Connect Partner membership)

**Closed (do not propose):**
- **Path 2c — retail funds-routing.** PSRs 2017 reg.138 / PSD2 / 18 USC §1960 / Suriname WTK 2011 — unauthorised payment service. (ADR-005)
- **Fork 1 — data redistribution.** RELX/LSEG-scale incumbents + commodity floor. (ADR-004)
- **Fork 2 — VoP/BAV specialist.** iPiD/SurePay incumbency, EU IPR window already closed. (ADR-004)
- **Path 4(c) — autonomous transactional MCP.** 24–36-month research bet; Swift has no agent identity model + PSD2/OFAC strict liability. (ADR-005)
- **Path 5 BaaS-style sponsor variant.** Collapses to PI/EMI requirement; Synapse/Solid/Bond cautionary precedent. (ADR-005)

**Design defaults:**
- **Permissioning:** open decision — candidates include OAuth 2.1 + PKCE (Plaid/Narmi pattern, human-rooted), OAuth + RFC 9396 RAR (fine-grained per-call scoping for Bucket B), DPoP (sender-constrained tokens), Dynamic Client Registration + mTLS (per-agent identity), and SPIFFE/SPIRE (self-hosted workload identity). The agent-identity dimension matters: OAuth 2.0 alone was designed for human-delegated access. Bucket B's human-approval gate must be cryptographically bound to the approving operator, not just to the agent's session.
- **Write semantics:** read-propose-approve (Slash pattern) for Bucket B — never autonomous-transactional
- **Architecture:** multi-source-neutral integration layer (the moat against any single data provider shipping their own MCP wrapper)

If a suggestion drifts toward a closed item, stop and flag it as a closed decision rather than proposing — name the ADR being challenged before pivoting.

---

## Current Stack (Track 2)

| Component | Detail |
|---|---|
| Dev host | gentoo-x13 (Gentoo Linux) |
| Staging host | Windows home lab |
| Prod/demo host | Hosting provider |
| GitHub | mblake4u |
| Cursor workspace | ~/dev/Cursor/swift-token-server.code-workspace |
| Start stack | `cd ~/dev/github/mblake4u/swiftops && ./docker_start_both_servers.sh` |

### Containers

**swift-token-server** (`everyday-ai/swift-token-server:v0.3.2-dev`, port 82)
- Flask + gunicorn
- `GET /token` returns OAuth Bearer token from sandbox.swift.com via RFC 7523 JWT-Bearer
- `GET|POST|... /proxy/<path>` — CORS proxy with token caching
- `X-SWIFT-Signature` header on mutating proxy requests (POST/PUT/PATCH/DELETE) — v0.3.2-dev
- Secrets required at runtime (NOT in git): `.env`, `Sandbox-Certificate.pem`, `Sandbox-Privatekey.pem`

**swift-swagger-ui** (`everyday-ai/swift-swagger-ui:v1.1.0-dev`, port 83)
- nginx
- Swift Messaging API v2.1.0 spec baked in at build time

**swift-ui-client** (`everyday-ai/swift-ui-client:v0.1.0-dev`, port 84)
- FastAPI + HTMX + Jinja2
- Distributions list (live, auto-refreshing) and detail pages
- JSON API endpoints at `/api/*` for future React migration
- Env vars: `PROXY_BASE_URL`, `TOKEN_SERVER_URL`, `SWAGGER_UI_URL`

**swift-mcp-gateway** (`everyday-ai/swift-mcp-gateway:v0.1.0-dev`, port 85)
- MCP server exposing Swift Messaging API endpoints as MCP tools (wrapping the token server's `/proxy` route), consumable from Claude Desktop / Claude Code / Agents
- Env vars: `PROXY_BASE_URL` (points at token server's `/proxy` route)
- Depends on `swift-token-server`
- **Convergence point of Track 2 and Track 3** — this is the seed of the Fork 3 product surface

### Auth Context (Sandbox)

| Field | Value |
|---|---|
| Scope | `swift.alliancecloud.api` (no suffix) |
| `aud` | `sandbox.swift.com/oauth2/v1/token` (no `https://` prefix) |
| `sub` | Must match CN in sandbox certificate |
| JWT header | Must include `x5c` |
| Credentials | From Swift Developer Portal |

> ⚠️ **Always use sandbox settings. Flag immediately if production settings appear.**

### CORS Constraint

Browsers cannot call `sandbox.swift.com` directly. Swagger UI generates correct curl commands — run those in a terminal. Option C (proxy route) is the planned permanent fix.

### Branch + Image Tag Model

| Branch | Environment | Image tag |
|---|---|---|
| `dev` | gentoo-x13 | `-dev` |
| `staging` | Windows home lab | `-staging` |
| `main` | Hosting provider | `YYYYMMDD` |

---

## Roadmap

### Completed (Track 2 foundation)
1. Option C — `/proxy` endpoint + token caching ✓
2. Python UI client (FastAPI + HTMX, port 84) ✓
3. Smoke tests, ADR template, open source hygiene (LICENSE, CONTRIBUTING, README) ✓
4. GitHub Actions CI — pip-audit + docker build + smoke tests on every push ✓
5. FastAPI evaluation — selected for UI client ✓
6. OSS landscape baseline + monthly scans ✓
7. `pip-audit` baseline ✓
8. `X-SWIFT-Signature` for mutating proxy requests (v0.3.2-dev) ✓
9. swift-mcp-gateway scaffold (v0.1.0-dev, port 85) ✓
10. swift-mcp-gateway deployed and tested in staging (Windows home lab, 2026-05-01) ✓

### Open — Track 2
- **Supply chain security:** hash pinning + CI integration (pip-audit baseline done)
- swift-mcp-gateway hardening — auth, eval methodology, additional tools
- Prod/demo deployment (hosting provider)

### Open — Track 3 (Fork 3)
- Define Fork 3 MVP scope (next dedicated thread; see [Fork 3 — MCP Integration Layer Concept](https://www.notion.so/352aa5f988dd814ca819f0e89e68fe52))
- Resolve open decisions: pack first-or-hybrid, auth/tenancy model, pricing, deployment model, Fork 3↔Track 2 relationship, eval methodology
- SCB outreach: identify LinkedIn contact, draft first-message variants, request 30-min discovery call
- Decide OSS-vs-commercial trigger criteria
- Update claude.ai Project system prompt with Track 3 / Fork 3 context

Track 3 direction is governed by **ADR-004** (Fork 3 selection), **ADR-005** (Path 2c blocked, Path 4(c) deferred, Path 5 BaaS variant blocked), and **ADR-006** (SCB anchor, DSB out of scope).

---

## Key Principles

### Search Before Building
Always check for existing open-source AND commercial tools before implementing. Document what was found and why build vs. reuse. OSS coverage at the [Open-Source Landscape](https://www.notion.so/34baa5f988dd813e8974ee55e5bbfd7e) Notion page; commercial coverage at [Pack C / Pack A — Competitive Landscape](https://www.notion.so/352aa5f988dd81c4ba42ffbe971b0f54).

### Open Source First (Track 2); Undecided (Track 3)
**Track 2** (dev infra) — all tools published as open source unless there's a specific reason not to. Every repo needs: `LICENSE`, `CONTRIBUTING.md`, issue templates, `README`.

**Track 3** (Fork 3) — OSS-vs-commercial end-state is undecided (50/50 framing). Design choices must keep both end-states viable until the SCB design-partner conversation closes. Don't bake in commercial-only assumptions; don't bake in OSS-only assumptions either.

### Prototype Discipline
- ADRs for key architectural decisions (state decision, context, options, rationale)
- 2–3 smoke tests per service
- Devlog / write-up per milestone
- GitHub Actions CI on every repo

### Evaluate Alternatives
Current stack is not fixed. Regularly surface better languages, frameworks, or architectural approaches.

### Honour Closed Decisions
A list of paths/forks/customers has been explicitly closed (see Track 3 — Product Direction above). If a suggestion drifts toward any of them, stop and flag it as a closed decision rather than proposing — name the ADR being challenged before pivoting. New evidence can reopen a decision; session-level reasoning cannot.

---

## Development Defaults

- **Language:** Python (unless otherwise specified)
- **Environment:** Sandbox always
- **API reference:** Swift Messaging API v2.1.0 OpenAPI spec is the definitive source for all endpoints and request/response formats
- **Error handling:** Always include in code examples
- **Architecture changes:** Frame as an ADR

---

## Repo Structure

```
~/dev/github/mblake4u/
├── swiftops/                          ← this repo: orchestration + project context
│   ├── CLAUDE.md                      ← this file
│   ├── docker-compose.yml             ← orchestrates all three containers
│   ├── docker_start_both_servers.sh   ← start the full stack
│   └── dashboard*.png, detail*.png    ← screenshots
├── swift-token-server/                ← Flask token + proxy service (git repo)
│   ├── token_server.py                ← main app: /token, /proxy, /health
│   ├── tests/
│   │   ├── conftest.py
│   │   └── test_smoke.py
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── docker-compose.yml
│   ├── apprunner.yaml
│   ├── .env                           ← runtime secret, gitignored
│   ├── Sandbox-Certificate.pem        ← runtime secret, gitignored
│   └── Sandbox-Privatekey.pem         ← runtime secret, gitignored
├── swift-swagger-ui/                  ← Swagger UI for API spec (git repo)
│   ├── SWIFT-API-Swift-Messaging-2.1.0-swagger.yaml  ← definitive API spec
│   ├── swagger-initializer.js
│   └── index.html
├── swift-mcp-gateway/                 ← MCP server exposing Swift APIs as MCP tools (git repo)
│   ├── app/
│   │   └── main.py                    ← FastMCP server; Streamable HTTP transport at /mcp
│   ├── docs/
│   ├── tests/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── LICENSE
│   ├── CONTRIBUTING.md
│   └── README.md
└── swift-ui-client/                   ← FastAPI + HTMX dashboard (git repo)
    ├── app/
    │   ├── main.py                    ← FastAPI app: JSON API + HTMX partials + pages
    │   ├── static/style.css
    │   └── templates/
    │       ├── base.html
    │       ├── index.html
    │       ├── distribution.html
    │       └── partials/
    │           ├── status.html
    │           └── distributions.html
    ├── docs/ADR-001-python-ui-client-stack.md
    ├── tests/
    │   ├── conftest.py
    │   └── test_smoke.py
    ├── Dockerfile
    └── requirements.txt
```

## Running the Stack

```bash
cd ~/dev/github/mblake4u/swiftops && docker compose up -d
```

Health checks:
- Token server: `curl http://localhost:82/health`
- Proxy test: `curl http://localhost:82/proxy/distributions`
- Swagger UI: open `http://localhost:83` in browser
- Dashboard: open `http://localhost:84` in browser
- MCP gateway: connect from Claude Desktop / Claude Code as an MCP client at `http://localhost:85/mcp` (Streamable HTTP transport). Liveness only: `docker ps --filter name=swift-mcp-gateway`.

## Tooling

| Tool | Role |
|---|---|
| Claude Code in Claude Desktop | Primary coding assistant |
| Cursor | Fallback editor |
| Claude Cowork | Task automation |
| MCPs | Build for Claude first, evaluate OpenClaw later |
| Docker | All services |

### Active MCPs

| MCP | Used for |
|---|---|
| Notion | Read/write runbooks directly from Claude Code |
| Slack | Send updates, search conversations |
| Gmail | Draft/search email |
| Google Drive | Read/write files |
| Supabase | Database operations (future) |
| Vercel | Deploy, check logs (future) |
| Postman | API collection management |
| Scheduled Tasks | Cron jobs for Claude agents |
| Playwright | Browser automation + screenshot testing of the dashboard |

---

## Known Gotchas

- **npm on Gentoo + sing-box VPN:** npm's Node.js HTTP client doesn't route through the sing-box tproxy rules that curl uses. To install npm packages, temporarily set the proxy: `npm config set proxy socks5://127.0.0.1:1081 && npm config set https-proxy socks5://127.0.0.1:1081`. Clear afterwards: `npm config delete proxy && npm config delete https-proxy`. Global installs also require sudo or use `--prefix ~/.local`.
- **Playwright MCP:** Installed at `~/.local/node_modules/@playwright/mcp/cli.js`. Configured in `~/.config/Claude/claude_desktop_config.json`. Use for screenshot/click testing of the dashboard at `http://localhost:84`. The "Control Chrome" connector (macOS-only/AppleScript) and "Claude in Chrome" extension are not reliable on Gentoo Linux.
- **Swift API distribution IDs:** `GET /distributions/{id}` may return a different ID than requested — sandbox behaviour, not a proxy bug.
- **Timezone bug (Windows):** `datetime.utcnow().timestamp()` produces incorrect epoch values on non-UTC machines. Use `time.time()` instead.
- **MGW profile management:** No PATCH endpoint — DELETE + recreate required.
- **Certificate API responses:** Private key is masked in MGW API responses but is present; expected behaviour.
- **JWT `x5c` header:** Mandatory — do not omit.
- **Scope in JWT claim:** Use `swift.alliancecloud.api` (without `/access_to_service` suffix).

---

## Security Rules

- **Never** expose API keys, secrets, consumer keys, keystore passwords, or private keys in responses or commits
- `.env`, `Sandbox-Certificate.pem`, `Sandbox-Privatekey.pem` are runtime secrets — not in git

---

## Project Docs

Runbooks and project context live in Notion under **Swift Lab** (the parent page; renamed from "Swift Microgateway Lab" during the 2026-04-23 pivot). Master narrative lives in **Project Vision & Goals**.

| Page | ID | Role |
|---|---|---|
| [Swift Lab](https://www.notion.so/32caa5f988dd81279a53e570b2e74655) | `32caa5f9-88dd-8127-9a53-e570b2e74655` | Parent / navigation hub |
| [Project Vision & Goals](https://www.notion.so/34baa5f988dd814d94f4e16190ccecc5) | `34baa5f9-88dd-814d-94f4-e16190ccecc5` | Master narrative, three-tracks structure |
| [ADR Register](https://www.notion.so/34caa5f988dd81d98475cf78440828cc) | `34caa5f9-88dd-81d9-8475-cf78440828cc` | All ADRs (latest: 004/005/006 from 2026-04-30) |
| [Project Pivot 2 — Path & Fork Direction](https://www.notion.so/352aa5f988dd8130a53bc27473c56b46) | `352aa5f9-88dd-8130-a53b-c27473c56b46` | 2026-04-30 pivot narrative |
| [Commercialisation Strategy](https://www.notion.so/352aa5f988dd818aa719c74e1cf14885) | `352aa5f9-88dd-818a-a719-c74e1cf14885` | Track 3 parent; Path × Entity matrix |
| [Fork 3 — MCP Integration Layer Concept](https://www.notion.so/352aa5f988dd814ca819f0e89e68fe52) | `352aa5f9-88dd-814c-a819-f0e89e68fe52` | Product scaffold + Open Decisions list |
| [Pack C / Pack A — Competitive Landscape](https://www.notion.so/352aa5f988dd81c4ba42ffbe971b0f54) | `352aa5f9-88dd-81c4-ba42-ffbe971b0f54` | Commercial competitor map |
| [SCB Anchor Customer Dossier](https://www.notion.so/352aa5f988dd81569c41dc6918292054) | `352aa5f9-88dd-8156-9c41-dc6918292054` | Anchor profile + outreach scaffold |
| [Deep Research Appendix](https://www.notion.so/352aa5f988dd819ebf99e66f493d50fe) | `352aa5f9-88dd-819e-bf99-e66f493d50fe` | Per-path detail, API inventory, lawyer questions |

Use the IDs directly with the Notion MCP (`notion-fetch`) rather than re-searching. For pages not listed here (Open-Source Landscape, Standards & Regulatory Watch, Market Intelligence, Product Signals, Zero-Footprint Docker Runbook, Pivot 1, etc.), navigate from the **Swift Lab** parent.
