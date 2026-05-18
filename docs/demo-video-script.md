# SwiftOps demo video — 75-second script

Target length: **75 seconds**. Target audience: a recruiter or fellow engineer scanning the repo cold; needs to see "this is real, this works, this is interesting" in under 90 seconds.

## Recording setup

| Spec | Recommendation |
|---|---|
| Tool | **Loom** (easiest — cloud upload, share link, no editing) OR **OBS Studio** + manual cut (more control, free) OR **ScreenStudio / Screen.Studio** (macOS, polished but paid) |
| Resolution | 1920×1080 minimum |
| Audio | Voiceover (optional but recommended — adds personality). Built-in mic is fine; quiet room more than gear |
| Cursor | Make it clickable and visible (Loom default is good) |
| Browser zoom | 110–125% so text is legible at 1080p video resolution |
| Pre-record | Have all four containers running, Claude Desktop open with MCP gateway connected, dashboard at localhost:84 in a tab |

## Script — 75 seconds, 5 cuts

### Cut 1 — Hook (0:00–0:08) — 8s
**On screen:** swiftops/README.md heading at the top of GitHub.
**Voiceover:** *"SwiftOps. A zero-footprint Swift Alliance Cloud API tooling stack — built solo over three months, closed out as an open-source portfolio piece. Here's what it does."*

### Cut 2 — Dashboard (0:08–0:25) — 17s
**On screen:** Switch to `http://localhost:84/` — the HTMX dashboard. Hover over a couple of distributions. Click into a distribution detail.
**Voiceover:** *"FastAPI + HTMX dashboard, live distributions from the Swift sandbox, auto-refreshing. Click into a distribution and you can see its routing metadata. All authenticated through a token-server proxy that handles OAuth, JWT-Bearer, and X-SWIFT-Signature transparently."*

### Cut 3 — MCP gateway in Claude (0:25–0:50) — 25s
**On screen:** Switch to Claude Desktop. Have a prompt ready: *"List the Swift distributions"*. Submit it. Show the MCP tool call expanding to `list_distributions`, then the response. Then a second prompt: *"Show me the FIN message body for distribution [paste an ID]"*. Show the second tool call.
**Voiceover:** *"The agent-facing piece is an MCP gateway — ten Swift API operations exposed as Claude tools. So I can just ask Claude — 'list the Swift distributions' — and it picks the right tool, populates the args, and surfaces the data. Same pattern for downloading FIN messages, sending MX, ACK or NAK."*

### Cut 4 — Eval methodology (0:50–1:08) — 18s
**On screen:** Switch to the Notion methodology page ([https://www.notion.so/Evaluating-an-MCP-Gateway-A-Methodology-364aa5f988dd8038bb21d34880ca6eab](https://www.notion.so/Evaluating-an-MCP-Gateway-A-Methodology-364aa5f988dd8038bb21d34880ca6eab)). Scroll to the Run A pass-rates table briefly. Then scroll to the five-cluster analysis.
**Voiceover:** *"Before closing the project out I shipped a published evaluation methodology for the MCP gateway — three orthogonal axes, multi-model, prompt-cached. The first baseline surfaced five distinct kinds of issue. One of them — a real product bug — was caught by the latency probe and missed by the model-eval axes. The methodology rigour is the artefact that travels."*

### Cut 5 — End slate (1:08–1:15) — 7s
**On screen:** swiftops repo URL or the Substack post URL. Static frame.
**Voiceover:** *"Full write-up on Substack at mblake4u.substack.com. Code at github.com/mblake4u/swiftops. Successor project in payments technology, more to come."*

## After recording

1. Upload to YouTube as **Unlisted**, OR keep on Loom and grab the share link.
2. Paste the URL back into Claude Code and ask to wire it into the README ("update the placeholder in swiftops/README.md").
3. Optional: paste the URL into the Substack post too (Substack supports YouTube embeds inline).
4. Optional: include the URL on the LinkedIn / X posts when published.

## Optional — additional screenshots worth capturing

These would round out the README's "See it work" section if you have a few extra minutes:

- **Claude Desktop + MCP gateway** in the middle of a conversation (cut 3 above, paused at a good frame). Save as `docs/screenshots/mcp-in-claude.png`.
- **Swagger UI** at `localhost:83` showing the Swift Messaging API v2.1.0 spec expanded. Save as `docs/screenshots/swagger.png`.
- **Notion methodology page** — the five-cluster section at full screen. Save as `docs/screenshots/methodology-clusters.png`.

If you capture any of these, drop them into `docs/screenshots/` with the suggested filenames and the README can pick them up automatically.

## What "good enough" looks like

A 75-second video uploaded to YouTube unlisted is good enough. Don't over-polish. The point is to demonstrate "this works" — not to compete with marketing-team production values. First take + light trim is the right amount of effort.
