# Round 3 (Nova, Oct 11 9 PM): judge-ready polish. Local only. Never deploy, never wrangler login/deploy, never git push.

## 1. /demo redesign (this is what the demo video films; it must look like a real product, 1920x1080 and 390px)
Current page is a narrow plain column. Redesign src/ui/demo-html.js (still one self-contained file, no CDN, no web fonts download; system font stack):
- Layout at >=1200px: left "device" pane styled like a smart display (rounded dark card, large readable answer text, subtle listening/thinking state indicator, the user's question echoed above the answer), right pane "What Board Clerk did" = live MCP trace as a readable timeline (method, tool name, key args, latency ms, result summary; expandable raw JSON). At <900px stack vertically.
- Under the answer, a structured decision card when a tool returns a motion: motion text, outcome badge (passed/failed/tabled), tally, mover / seconder each with a status chip (Spoken in recording / Confirmed by clerk / Needs confirmation), "Listen at 00:22:10" timestamp chip, meeting date.
- Pending confirmations view: list with one-click "Confirm <AI suggestion>" buttons and a roster dropdown (calls confirm_mover_seconder).
- Minutes view: rendered markdown (simple safe renderer, textContent based, headings/bullets/bold only) with an "Approve minutes" button; show the refusal message prominently when the guardrail blocks, and an approved banner after.
- Suggested prompts as chips: "What did we decide about the roof?", "Who seconded the reserve study?", "What still needs confirming?", "Draft last night's minutes", "Approve the minutes".
- Persistent small banner: "Simulated Alexa+ experience · synthetic fictional board · not an Alexa device". Keep: no speechSynthesis, mic = speech recognition input only.
- Add `?board=<id>` URL param to reuse a board, and `?reset=1` to start a fresh isolated board (for clean video takes). Add data-testid attributes on key elements for Playwright.
- Include an "Ingest last night's meeting" action that ingests demo/juniper-2026-10-15-transcript.txt (embed the transcript text in the page bundle or serve it at GET /demo/transcript.txt) then shows motions + pending.
- Accessibility: labels, focus rings, aria-live on the answer, contrast AA.
Update the /demo test and add tests for /demo/transcript.txt if added.

## 2. Version + health
Bump to 0.3.0 (package.json, server info, health JSON — health must read the same constant, not a separate hardcoded string).

## 3. Judge docs (truthful; no claims we did not verify)
- docs/JUDGES.md: 2-minute test path (open /demo; or Inspector CLI commands that work against https://lumegrid-board-clerk-v02.lumegridai.workers.dev/mcp: tools/list, start_demo_board, what_did_we_decide, list_pending_confirmations, confirm, request+approve, get_audit_log), plus how to run locally (npm i, npm test, npx wrangler dev). State that judge boards are isolated by board_id.
- docs/FRICTION-LOG.md and docs/PRODUCT-FEEDBACK.md drafts using ONLY these real observations from our Oct 5 and Oct 11 build (date each; mark anything not reproduced as "observed once"):
  a) @modelcontextprotocol/server 2.3.1 createMcpHandler `route` option did not restrict paths: tools executed on POST / and /mcp/ too; we added our own routing (REVIEW-FINDINGS H2).
  b) The handler gives no built-in Origin validation even though the 2025-11-25 transport spec requires servers to validate Origin; we wrote an allowlist and found our first version was DNS-rebinding-bypassable (H1).
  c) Stateless Streamable HTTP means cross-session memory must live elsewhere; we used a SQLite-backed Durable Object per board. No Alexa+ guidance on where per-household state should live for a self-hosted MCP server.
  d) Event rules allow a self-hosted MCP server or a simulated experience, but we found no way to connect a third-party remote MCP server to a real Alexa+ device during the event, so we built the simulated web experience. (Same doc-reconciliation pain as our LiftCheck feedback.)
  e) MCP Inspector CLI worked for tools/call against the remote server; passing a JSON array argument (roster) worked; extracting ids from text content was awkward, structuredContent fixed it.
  f) Wrangler 3.114 printed an upgrade-to-4 warning; on a headless box the OAuth login did not survive a machine restore and an API token was needed.
  Each item: what happened, impact (time lost estimate labelled as estimate), what would have helped. Product feedback per tool used: MCP TypeScript server SDK, MCP Inspector, Cloudflare Workers + Durable Objects, Wrangler, (Alexa+ docs). Keep factual.
- README: link JUDGES.md, keep honest accuracy numbers; mention v0.3.0.

## 4. Repo readiness
LICENSE (MIT, "LumeGrid"), .gitignore (node_modules, .wrangler, .dev.vars), package.json metadata (description, repository placeholder removed, engines node>=20, license), .github/workflows/ci.yml running npm ci && npm test on node 20, docs/ARCHITECTURE.md with a mermaid diagram (client -> Worker /mcp -> McpServer -> engine -> Durable Object per board). Remove stale files that would confuse judges (e.g. duplicate fixtures .json if unused by code — check imports first). No secrets anywhere (grep).

Finish: npm test, test:engine, test:mcp-local pass. Print summary.
