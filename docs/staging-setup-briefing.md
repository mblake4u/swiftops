# SwiftOps Staging Setup — Windows Home Lab

## Context

This is the SwiftOps project: open-source Swift Alliance Cloud API tooling.
You are setting up the **staging environment** on a Windows machine that has
Docker Desktop, GitHub Desktop, and Claude Desktop already installed.

The stack is four Docker containers:

| Container | Image | Port |
|---|---|---|
| swift-token-server | everyday-ai/swift-token-server:v0.3.2-staging | 82 |
| swift-swagger-ui | everyday-ai/swift-swagger-ui:v1.1.0-staging | 83 |
| swift-ui-client | everyday-ai/swift-ui-client:v0.1.0-staging | 84 |
| swift-mcp-gateway | everyday-ai/swift-mcp-gateway:v0.1.0-staging | 85 |

All repos are public on GitHub under `mblake4u`.

---

## Step 1 — Choose a working directory

All repos must be siblings in the same parent folder. Suggested:

```
C:\repos\github\mblake4u\
```

Create it if it does not exist:

```powershell
New-Item -ItemType Directory -Force -Path C:\repos\github\mblake4u
```

---

## Step 2 — Clone all repos

```powershell
cd C:\repos\github\mblake4u
git clone https://github.com/mblake4u/swift-token-server.git
git clone https://github.com/mblake4u/swift-swagger-ui.git
git clone https://github.com/mblake4u/swift-ui-client.git
git clone https://github.com/mblake4u/swift-mcp-gateway.git
git clone https://github.com/mblake4u/swiftops.git
```

---

## Step 3 — Place the runtime secrets

Three secret files must be copied into `C:\repos\github\mblake4u\swift-token-server\` before
the stack will start. The user will provide these — do not proceed until they
are in place:

```
C:\repos\github\mblake4u\swift-token-server\.env
C:\repos\github\mblake4u\swift-token-server\Sandbox-Certificate.pem
C:\repos\github\mblake4u\swift-token-server\Sandbox-Privatekey.pem
```

Verify they exist:

```powershell
Test-Path C:\repos\github\mblake4u\swift-token-server\.env
Test-Path C:\repos\github\mblake4u\swift-token-server\Sandbox-Certificate.pem
Test-Path C:\repos\github\mblake4u\swift-token-server\Sandbox-Privatekey.pem
```

All three must return `True` before continuing.

---

## Step 4 — Check out staging branches and build Docker images

Each repo has a `staging` branch. Check it out before building so the images
correspond to the correct branch per the project's branch/tag model.

```powershell
cd C:\repos\github\mblake4u\swift-token-server;  git checkout staging
cd C:\repos\github\mblake4u\swift-swagger-ui;    git checkout staging
cd C:\repos\github\mblake4u\swift-ui-client;     git checkout staging
cd C:\repos\github\mblake4u\swift-mcp-gateway;   git checkout staging
```

Then build each image. Docker Desktop must be running.

```powershell
cd C:\repos\github\mblake4u\swift-token-server
docker build -t everyday-ai/swift-token-server:v0.3.2-staging .

cd C:\repos\github\mblake4u\swift-swagger-ui
docker build -t everyday-ai/swift-swagger-ui:v1.1.0-staging .

cd C:\repos\github\mblake4u\swift-ui-client
docker build -t everyday-ai/swift-ui-client:v0.1.0-staging .

cd C:\repos\github\mblake4u\swift-mcp-gateway
docker build -t everyday-ai/swift-mcp-gateway:v0.1.0-staging .
```

Verify all four images exist:

```powershell
docker images | findstr everyday-ai
```

Expected output — four lines, one per image with `-staging` tag.

---

## Step 5 — Start the stack

```powershell
cd C:\repos\github\mblake4u\swiftops
docker compose -f docker-compose.staging.yml up -d
```

Verify all four containers are running:

```powershell
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

Expected: four containers, all `Up`, ports 82–85 mapped.

---

## Step 6 — Smoke test each service

```powershell
# Token server health
curl http://localhost:82/health

# Proxy (will return Swift sandbox data)
curl http://localhost:82/proxy/distributions

# Swagger UI — open in browser
Start-Process "http://localhost:83"

# Dashboard — open in browser
Start-Process "http://localhost:84"

# MCP gateway handshake
curl -X POST http://localhost:85/mcp `
  -H "Content-Type: application/json" `
  -H "Accept: application/json, text/event-stream" `
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"test","version":"0.1"}}}'
```

The MCP handshake should return a JSON response containing `"name":"Swift API Gateway"`.

---

## Step 7 — Configure Claude Desktop

The Claude Desktop config file is at:

```
%APPDATA%\Claude\claude_desktop_config.json
```

Typically: `C:\Users\<username>\AppData\Roaming\Claude\claude_desktop_config.json`

Read the current file, then add the `swift-api-gateway` entry to the
`mcpServers` object if it is not already present:

```json
"swift-api-gateway": {
  "command": "C:\\Program Files\\nodejs\\npx.cmd",
  "args": ["-y", "mcp-remote", "http://localhost:85/mcp"]
}
```

This uses the `mcp-remote` package (launched via `npx`) as a stdio bridge that
proxies to the HTTP MCP server on port 85. The URL-only schema
(`{"url": "..."}`) is rejected by current Claude Desktop versions.

On Windows, the `.cmd` suffix on `npx.cmd` is required — Claude Desktop spawns
processes via Node's `child_process` which doesn't auto-resolve `.cmd`
extensions. Prerequisite: Node.js installed (provides `npx`).

After editing, restart Claude Desktop. It should load without MCP errors and
the Swift API tools (`list_distributions`, `get_distribution`, etc.) will be
available.

---

## Step 8 — Verify MCP in Claude Desktop

After restarting Claude Desktop, open a new conversation and ask:
> "Call list_distributions"

It should return live distribution data from the Swift sandbox.

---

## Troubleshooting

| Symptom | Check |
|---|---|
| Container exits immediately | `docker logs swift-token-server` — likely missing `.env` or cert files |
| Port conflict | Another process on 82–85; `netstat -ano \| findstr :82` |
| MCP not loading in Claude Desktop | Confirm port 85 is up: `curl http://localhost:85/mcp`. Also check `%APPDATA%\Claude\logs\main.log` for `Skipped invalid MCP server config entries` warnings — if present, the config schema is being rejected and you need the stdio bridge form (Step 7). |
| Token error in proxy | Check `.env` values match Swift Developer Portal sandbox credentials |
