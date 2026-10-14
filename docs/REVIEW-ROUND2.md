# Round 2 (Nova, Oct 11 2026): restore + red-team + simulated Alexa+ UI
Do NOT deploy, log in, or touch Cloudflare. Local only. Background spec: docs/SPEC-v0.2.md.

Round 4 superseded two instructions below. `wrangler.toml` name is `lumegrid-board-clerk-v02`, not `lumegrid-board-clerk-mcp`. Writes on `juniper-hollow-demo` are refused. The accuracy sentence in section D is not the current README: those ratios are a separate pipeline test, not this extractor, and this server does not call Gemini.

## 0. Restore (a box rollback lost these from the v0.2 build; recreate them)
- package.json scripts: "test": "node --test test/", "test:engine", "test:mcp-local" (existing), "test:remote": "node scripts/smoke-remote.mjs". Version 0.2.x.
- wrangler.toml: keep name "lumegrid-board-clerk-mcp"; add [[durable_objects.bindings]] name="BOARD_STATE" class_name="BoardState" and [[migrations]] tag="v1" new_sqlite_classes=["BoardState"]; drop the AMAZON_SUBMIT var. src/index.js must export BoardState.
- test/mcp.test.mjs: drive the real Streamable-HTTP handler in-process (Request objects; Accept: application/json, text/event-stream; parse JSON or SSE). Cases: initialize with protocolVersion 2025-11-25 and assert it; tools/list has all tools with titles + annotations; resources/templates/list; prompts/list; ingest_meeting -> list_pending -> approve_minutes refused -> confirm both -> approve ok; CROSS-SESSION: a fresh server instance sharing the same MemoryStore sees the confirmation and ingested meeting; board isolation; input limits rejected.
- scripts/smoke-remote.mjs <baseUrl>: same flow over real HTTP with a random board_id, then reset_board.
- README rewrite (see D).

## A. Independent review
Review src/ as a hostile hackathon judge and a security reviewer. Write docs/REVIEW-FINDINGS-1011.md (severity, file:line, fixed?). Fix all High/Medium.

## B. Known issues
1. ingest_meeting output is generic (org "Board (FICTIONAL)", title "Board meeting <date>", agenda_item "Business item N"). Derive agenda items from chair cues ("next item", "item N", "moving on to", "new business:", "old business:"); org/title from args or '#' header lines; meeting_id slug from org+date when org given.
2. Tallies: members absent at roll call must not count as ayes on voice votes. Test it.
3. confirm_mover_seconder: value must match a roster member (full/first/last, case-insensitive) -> normalized full name; else isError listing the roster. Keep 80-char limit.
4. Default board "juniper-hollow-demo" is the public judged demo: reset_board on it restores fixtures-only state; responses on it include demo_board: true. Add tool start_demo_board {board_id} that copies the two Juniper fixtures into a new board's layered state so judges get an isolated full copy (and fixtures-backed queries work there).
5. Origin validation on /mcp (MCP 2025-11-25 DNS-rebinding guidance): if Origin present and not allowlisted (worker's own origin, http://localhost:*, http://127.0.0.1:*), 403. No Origin = allowed. Tests.
6. Reject /mcp POST bodies > 256 KB with 413.
7. Tool errors returned as tool results with isError and actionable messages.
8. what_did_we_decide top_answer: <= 2 natural sentences for a voice assistant, natural dates ("August 20th"), say amounts as written in motion, and say when mover/seconder is unconfirmed.

## C. Simulated Alexa+ web experience at GET /demo
Single self-contained HTML (no CDN, no build) served by the Worker (src/ui/demo-html.js exporting a string). Real MCP client over /mcp (initialize 2025-11-25, tools/call, JSON or SSE), collapsible MCP trace panel. Text input plus mic button using Web Speech *recognition* for INPUT only when available. NO speechSynthesis / no voice output (house rule). Simple deterministic intent routing: questions -> what_did_we_decide; "what needs confirming" -> list_pending_confirmations; "confirm <name> moved|seconded <motion>" -> confirm_mover_seconder; "draft minutes" -> draft_minutes; "approve minutes" -> request + approve. On first load calls start_demo_board with a random board id kept in localStorage. Clear labels: synthetic fictional data; simulated Alexa+ experience; not an Alexa device. Accessible (labels, focus, contrast, works at 390px). Test: GET /demo returns 200 text/html containing "2025-11-25".

## D. README
What it is; judge quick start (preview https://lumegrid-board-clerk-v02.lumegridai.workers.dev/mcp and /demo; `npx @modelcontextprotocol/inspector --cli <url> --transport http --method tools/list`); tools table; state model; guardrails; honest accuracy numbers (a separate MinutesDesk pipeline test on 5 public meetings reported 24/24 motions and outcomes and mover/seconder 16/24 and 14/24; those ratios are not this extractor); rule-based extractor is a $0 baseline for clean synthetic transcripts; synthetic data only. No old withdrawal language.

Finish: npm test, test:engine, test:mcp-local all pass. Print summary with test counts.
