# SwiftOps — Claude Code Context

> **Status (2026-05-18):** Closed out as an open-source portfolio piece. This file is the public technical reference for anyone cloning the repo to use Claude Code on it. Project narrative is in `docs/DEVLOG.md`.

---

## Project Overview

SwiftOps is a personal R&D project: a zero-footprint Swift Alliance Cloud API tooling stack (token server + Swagger UI + UI client + MCP gateway), with a published evaluation methodology for the MCP gateway.

See `docs/DEVLOG.md` for the full project story and what was built, learned, and decided across the three-month build.

---

## Current Stack

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

**swift-mcp-gateway** (`everyday-ai/swift-mcp-gateway:v0.1.1-dev`, port 85)
- MCP server exposing Swift Messaging API endpoints as MCP tools (wrapping the token server's `/proxy` route), consumable from Claude Desktop / Claude Code / Agents
- Ships with a [published evaluation methodology](https://www.notion.so/Evaluating-an-MCP-Gateway-A-Methodology-364aa5f988dd8038bb21d34880ca6eab) (ADR-003 in the gateway repo)
- Env vars: `PROXY_BASE_URL` (points at token server's `/proxy` route)
- Depends on `swift-token-server`

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

Browsers cannot call `sandbox.swift.com` directly. Swagger UI generates correct curl commands — run those in a terminal. The token server's `/proxy` route is the implemented fix.

### Branch + Image Tag Model

| Branch | Environment | Image tag |
|---|---|---|
| `dev` | gentoo-x13 | `-dev` |
| `staging` | Windows home lab | `-staging` |
| `main` | Hosting provider | `YYYYMMDD` |

---

## Key Principles

### Search Before Building
Always check for existing open-source AND commercial tools before implementing. Document what was found and why build vs. reuse.

### Open Source First
All tools published as open source. Every repo carries: `LICENSE`, `CONTRIBUTING.md`, issue templates, `README`.

### Prototype Discipline
- ADRs for key architectural decisions (state decision, context, options, rationale)
- 2–3 smoke tests per service
- Devlog / write-up per milestone
- GitHub Actions CI on every repo

### Evaluate Alternatives
Stack is not fixed. Regularly surface better languages, frameworks, or architectural approaches.

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
│   ├── CLAUDE.md                      ← this file (public technical reference)
│   ├── docker-compose.yml             ← orchestrates all four containers
│   ├── docker_start_both_servers.sh   ← start the full stack
│   ├── README.md                      ← project hub
│   ├── docs/
│   │   ├── DEVLOG.md                  ← full project narrative
│   │   └── screenshots/               ← curated portfolio screenshots
├── swift-token-server/                ← Flask token + proxy service (git repo)
│   ├── token_server.py                ← main app: /token, /proxy, /health
│   ├── tests/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── .env                           ← runtime secret, gitignored
│   ├── Sandbox-Certificate.pem        ← runtime secret, gitignored
│   └── Sandbox-Privatekey.pem         ← runtime secret, gitignored
├── swift-swagger-ui/                  ← Swagger UI for API spec (git repo)
│   ├── SWIFT-API-Swift-Messaging-2.1.0-swagger.yaml  ← definitive API spec
│   ├── swagger-initializer.js
│   └── index.html
├── swift-mcp-gateway/                 ← MCP server exposing Swift APIs as MCP tools (git repo)
│   ├── app/main.py                    ← FastMCP server; Streamable HTTP transport at /mcp
│   ├── docs/                          ← ADRs including the eval methodology
│   ├── evals/                         ← test set, harness, probe, results
│   ├── tests/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── LICENSE
│   ├── CONTRIBUTING.md
│   └── README.md
└── swift-ui-client/                   ← FastAPI + HTMX dashboard (git repo)
    ├── app/main.py                    ← FastAPI app
    ├── docs/ADR-001-python-ui-client-stack.md
    ├── tests/
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
| MCPs | Build for Claude first |
| Docker | All services |

---

## Known Gotchas

- **npm on Gentoo + sing-box VPN:** npm's Node.js HTTP client doesn't route through the sing-box tproxy rules that curl uses. To install npm packages, temporarily set the proxy: `npm config set proxy socks5://127.0.0.1:1081 && npm config set https-proxy socks5://127.0.0.1:1081`. Clear afterwards: `npm config delete proxy && npm config delete https-proxy`. Global installs also require sudo or use `--prefix ~/.local`.
- **Swift API distribution IDs:** `GET /distributions/{id}` may return a different ID than requested — sandbox behaviour, not a proxy bug.
- **Timezone bug (Windows):** `datetime.utcnow().timestamp()` produces incorrect epoch values on non-UTC machines. Use `time.time()` instead.
- **Certificate API responses:** Private key is masked in MGW API responses but is present; expected behaviour.
- **JWT `x5c` header:** Mandatory — do not omit.
- **Scope in JWT claim:** Use `swift.alliancecloud.api` (without `/access_to_service` suffix).

---

## Security Rules

- **Never** expose API keys, secrets, consumer keys, keystore passwords, or private keys in responses or commits
- `.env`, `Sandbox-Certificate.pem`, `Sandbox-Privatekey.pem` are runtime secrets — not in git
