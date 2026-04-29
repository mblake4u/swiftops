# SwiftOps — Claude Code Context

---

## Project Overview

SwiftOps is an open-source Swift Alliance Cloud API tooling initiative built on a zero-footprint Docker stack.

**Goals (in priority order):**
1. Build useful Swift API tooling, MCPs, and services
2. Technical portfolio: Solutions Architect / API+Integration Specialist / AI Tooling Engineer
3. Explore innovative Swift API use cases once foundation is solid

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

**swift-token-server** (`everyday-ai/swift-token-server:v0.3.1-dev`, port 82)
- Flask + gunicorn
- `GET /token` returns OAuth Bearer token from sandbox.swift.com via RFC 7523 JWT-Bearer
- `GET|POST|... /proxy/<path>` — CORS proxy with token caching
- Secrets required at runtime (NOT in git): `.env`, `Sandbox-Certificate.pem`, `Sandbox-Privatekey.pem`

**swift-swagger-ui** (`everyday-ai/swift-swagger-ui:v1.1.0-dev`, port 83)
- nginx
- Swift Messaging API v2.1.0 spec baked in at build time

**swift-ui-client** (`everyday-ai/swift-ui-client:v0.1.0-dev`, port 84)
- FastAPI + HTMX + Jinja2
- Distributions list (live, auto-refreshing) and detail pages
- JSON API endpoints at `/api/*` for future React migration
- Env vars: `PROXY_BASE_URL`, `TOKEN_SERVER_URL`, `SWAGGER_UI_URL`

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

## Current Roadmap (in order)

1. **Option C** — `/proxy` endpoint in `token_server.py` (CORS fix + token caching) `[done]`
2. **Python UI client** — 3rd container `[done]` — FastAPI + HTMX, port 84, v0.1.0-dev
3. **Smoke tests, ADR template, open source hygiene** `[done]` — smoke tests, ADR-001, README, LICENSE, CONTRIBUTING all shipped
4. **GitHub Actions CI** `[done]` — both repos; pip-audit + docker build + smoke tests on every push
5. **FastAPI evaluation** (candidate to replace Flask) `[done]` — FastAPI selected for UI client
6. **Search for existing Swift API open-source tools / MCP gateways** `[done]` — OSL baseline 2026-04-23; automated monthly scans active; `swiftinc/api-sample-code` Python dir found → revealed `X-SWIFT-Signature` gap → fixed in v0.3.2-dev
7. **Supply chain security** — `pip-audit` baseline `[done]`; hash pinning + CI integration `[not started]`
8. **`X-SWIFT-Signature` for mutating proxy requests** `[done]` — v0.3.2-dev

---

## Key Principles

### Search Before Building
Always check for existing open-source tools before implementing. Document what was found and why build vs. reuse.

### Open Source First
All tools published unless there's a specific reason not to. Every repo needs: `LICENSE`, `CONTRIBUTING.md`, issue templates, `README`.

### Prototype Discipline
- ADRs for key architectural decisions (state decision, context, options, rationale)
- 2–3 smoke tests per service
- Devlog / write-up per milestone
- GitHub Actions CI (after Option C)

### Evaluate Alternatives
Current stack is not fixed. Regularly surface better languages, frameworks, or architectural approaches. FastAPI is the current candidate to replace Flask.

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

Runbooks live in Notion under **Swift Microgateway Lab**. The current runbook reflects the live state of the project. Milestone snapshots are duplicated and frozen when a version ships.

| Version / Milestone | Notion |
|---|---|
| Current (live) | [Swift Microgateway Lab](https://www.notion.so/32caa5f988dd81279a53e570b2e74655) |
| v0.2 — Zero-Footprint Docker | Page ID `34aaa5f9-88dd-8137-a1e0-fb8cc1bf6819` |

When starting a new milestone: duplicate the current Notion page, rename it with the version/milestone label, and add a row to this table.
