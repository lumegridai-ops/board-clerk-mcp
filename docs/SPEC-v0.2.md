# Board Clerk MCP v0.2 build spec (Nova, Oct 11 2026)

Round 4 supersedes four lines in this file. The shared board `juniper-hollow-demo` is read-only for every write tool. `wrangler.toml` name is `lumegrid-board-clerk-v02`. The 24/24, 16/24, and 14/24 figures are a separate Gemini MinutesDesk pipeline test, not this extractor, and this repo does not call Gemini. `crosscheck` runs only when `ingest_meeting` receives `second_pass_motions`. The paragraphs below are the Oct 11 spec as written, except the withdrawal phrases were removed.

Target: Amazon "Build, Ship, Shape" Alexa+ track. Self-hosted MCP server, spec 2025-11-25, Streamable HTTP, on Cloudflare Workers.
Judges call "basic MCP wrapper around an existing API" obvious, and "agentic cross-service workflows with STATE ACROSS SESSIONS" creative.
Synthetic (FICTIONAL) data only. NO network calls to any LLM or email provider from this code. $0.

## Constraints
- Plain ESM JavaScript (no TypeScript build step). Node 20. Deps already installed: @modelcontextprotocol/server 2.3.1 (createMcpHandler, McpServer), zod 4, wrangler 3.114.
- Do NOT run `wrangler deploy`, `wrangler login`, or anything that touches Cloudflare or the network. Local only. Do not git push.
- Keep existing fixtures in src/fixtures/*.js unchanged (they are the seeded demo board).
- Tests use `node --test` (node:test + node:assert). All must pass offline with `npm test`.

## 1. Engine extraction -> src/engine/
Split current src/engine.js into a library (keep src/engine.js as a thin re-export so old imports still work):
- `src/engine/query.js`: the current list/get/decision/pending/confirm/whatDidWeDecide/draftMinutes logic, but operating on a meetings ARRAY passed in (not a module-global FIXTURES), so ingested meetings from state work too. Fix the bug where the stopword list in whatDidWeDecide drops "we" only via length filter; add simple stemming (roof/roofing, bid/bids) and synonym map (roof->reroof, pool->resurface); rank recent first on ties. Support date_from/date_to.
- `src/engine/validate.js`: zod port of MOTION_SCHEMA from /workspace/lumegrid/ventures/minutes-test/scripts/pipeline.py (fields agenda_item, motion_text, mover, seconder, outcome enum passed|failed|withdrawn|tabled|unclear, ayes, nays, abstain, absent, vote_method, motion_timestamp HH:MM:SS, vote_timestamp, confidence high|medium|low, uncertain_fields, evidence_quote). `validateMotions(json)` -> {ok, motions, errors}. Rejects bad timestamps, unknown outcomes.
- `src/engine/crosscheck.js`: faithful JS port of `crosscheck(draft, verify)`, `norm`, `tsec` from pipeline.py (pair by timestamp <=120s, flag disagreements on mover/seconder/outcome/nays, unclear, second-pass-only motions).
- `src/engine/namerule.js`: port of the spoken-name attribution rule (see /workspace/lumegrid/ventures/minutes-test/scripts/namerule.py): mover/seconder is SPOKEN only if the person's surname or first name (or alias) appears in transcript lines within -45s..+30s of the motion timestamp; otherwise status NEEDS CONFIRMATION with ai_suggestion and reason. Build aliases from roster names (first, last, full). Pure function: `attribute(motion, transcriptLines, roster)`.
- `src/engine/extract.js`: deterministic, $0 rule-based extractor for clean transcripts in the format `[HH:MM:SS] Speaker Name: text` (one per line). Detects: "I move/I'll move/move that/motion to ...", seconds ("second", "I'll second", "seconded"), chair-stated "moved by X, seconded by Y", votes ("all in favor", "opposed", roll-call "aye"/"nay" by named speakers, "motion carries/passes/fails/is withdrawn/tabled"), roll call/present. Produces motions in the validated schema with motion_timestamp/vote_timestamp, evidence_quote, confidence, uncertain_fields, then applies namerule. Must be honest: it is a baseline for clean synthetic transcripts only. This server does not call a model. A caller may pass a second motion list into ingest, which then runs crosscheck. Export `extractMeeting({transcript, roster, meeting_id, meeting_date, org, title})` -> meeting object in the same shape as fixtures (motions with id M1.., attribution, items minimal, present/absent from roll call).
- `src/engine/minutes.js`: draftMinutes renderer.

## 2. State across sessions -> Durable Object
- `src/state.js`: class `BoardState` (Durable Object, SQLite-backed; use `this.ctx.storage` KV-style get/put/list API which works on SQLite DOs). One instance per board_id (`env.BOARD_STATE.idFromName(board_id)`). Stores: ingested meetings, confirmations, approval requests, append-only audit log (who/what/when). Expose an internal JSON RPC via fetch (POST /op {op, args}).
- Also an in-memory `MemoryStore` with the identical interface for tests and for when the binding is absent.
- Default board_id "juniper-hollow-demo" is seeded from fixtures (fixtures are read-only base; confirmations/ingests layered on top).
- Limits: transcript <= 200 KB, <= 25 ingested meetings per board, board_id regex ^[a-z0-9-]{3,40}$, confirmation value <= 80 chars. `reset_board` clears a board's layered state (not fixtures).

## 3. MCP server -> src/index.js (+ src/server.js building the McpServer so tests can reuse it)
Server name "board-clerk", version "0.2.0". Every tool gets a `title` and `annotations` (readOnlyHint / destructiveHint / idempotentHint / openWorldHint=false). All tools accept optional `board_id`.
Tools:
- list_meetings, list_motions, get_motion, get_decision, what_did_we_decide (query, date_from, date_to, meeting_id), list_pending_confirmations, confirm_mover_seconder, draft_minutes (existing behavior, now state-aware)
- ingest_meeting {transcript, meeting_date, title?, org?, roster: [{name, role}]} -> runs extract.js, stores meeting, returns motions + pending confirmations. 
- request_minutes_approval {meeting_id, approver_name} -> records an approval request (status pending) with id; NEVER sends email; response text says it is recorded only.
- approve_minutes {meeting_id, approval_id, approver_name} -> GUARDRAIL: refuses (isError) while any mover/seconder is NEEDS CONFIRMATION; otherwise marks approved and returns final minutes markdown.
- get_audit_log {meeting_id?} -> audit entries.
- reset_board {} -> clears layered state for this board (destructiveHint true).
Tool results: return both `content` text (short spoken-style answer first line, then JSON) and `structuredContent` where practical.
Resources: `board://{board_id}/roster` and `board://{board_id}/meetings` (ResourceTemplate if available in this SDK version; otherwise static for the demo board).
Prompts: `ask_the_board` (arg: question) -> instructs the client to call what_did_we_decide and answer with motion, outcome, tally, mover/seconder status, and timestamp.
Health `GET /` JSON: name, version, mcp "/mcp", protocol "2025-11-25", transport streamable-http, synthetic true, tools list.
wrangler.toml: name stays as given by Nova later (do not invent deploy), add durable_objects binding BOARD_STATE class BoardState, and `[[migrations]] tag="v1" new_sqlite_classes=["BoardState"]`. Export BoardState from src/index.js. Remove the vars AMAZON_SUBMIT line.

## 4. Synthetic demo meeting
Add `demo/juniper-2026-10-15-transcript.txt`: FICTIONAL Juniper Hollow (same roster as fixtures) Oct 15 2026 meeting, ~60-90 lines, in the `[HH:MM:SS] Name: text` format: roll call, agenda approval, follow-up on the roof bid from Aug (e.g. change order), a motion where the mover is NOT named aloud (so it needs confirmation), a failed motion, a tabled motion, adjourn. Clearly mark FICTIONAL in a header comment line starting with '#' (extractor ignores # lines).

## 5. Tests (npm test runs all)
- test/extract.test.mjs: demo transcript -> expected motion count, outcomes, at least one NEEDS CONFIRMATION, timestamps valid per validate.js.
- test/namerule.test.mjs, test/crosscheck.test.mjs (port behavior cases), test/validate.test.mjs, test/query.test.mjs (roof query across Aug/Sep, date filter, synonyms).
- test/mcp.test.mjs: drive the real Streamable-HTTP handler in-process (createMcpHandler fetch with Request objects; Accept: application/json, text/event-stream; parse SSE or JSON). Cases: initialize with protocolVersion "2025-11-25" and assert negotiated version; tools/list includes all 14 tools with titles+annotations; resources/list or templates; prompts/list; ingest_meeting -> list_pending -> approve_minutes refused -> confirm both -> approve_minutes ok; CROSS-SESSION: a brand-new handler/server instance (new "session") sharing the same MemoryStore sees the confirmation and the ingested meeting; board isolation (board A confirm invisible on board B); input limits rejected.
- Keep scripts/smoke-engine.mjs and smoke-mcp-local.mjs working (update imports if needed).
- scripts/smoke-remote.mjs <baseUrl>: same flow over real HTTP against a deployed URL using a random board_id, then reset_board.
package.json scripts: "test": "node --test test/", "test:remote": "node scripts/smoke-remote.mjs".

## 6. README
Update README: what it is, tool table, state model, guardrail, honest accuracy numbers (a separate MinutesDesk pipeline test reported 24/24 motions and outcomes and mover/seconder 16/24 and 14/24; those ratios are not this extractor), rule-based extractor is a $0 baseline for synthetic demo, how to run tests, how to point MCP Inspector at /mcp. Remove the old withdrawal language (that decision is tracked outside the repo). No real people or orgs.
