# Board Clerk for Alexa+ — Devpost submission pack
Event: Amazon "Build, Ship, Shape" (Alexa+ track). Deadline **Fri Oct 23, 1:00 PM MDT**. Target submit Thu Oct 22.

## Verified Oct 14, 2026 ~10:35 AM MT
- /health -> v0.4.0, 16 tools; /demo 200; /mcp initialize OK (protocol 2025-11-25).
- scripts/smoke-remote.mjs against live host: PASS. `npm test`: 55/55.
- Video: ../board-clerk-video/out/board-clerk-demo.mp4 (166.1 s, 1080p30, captions burned + captions.srt). Script: ../board-clerk-video/SCRIPT.md.

## Links to paste
- Try it: https://lumegrid-board-clerk-v02.lumegridai.workers.dev/demo
- MCP endpoint: https://lumegrid-board-clerk-v02.lumegridai.workers.dev/mcp
- Video: <YouTube URL after upload>
- Repo: <GitHub URL after push>

## Devpost fields
**Name:** Board Clerk for Alexa+
**Tagline:** Ask your HOA board "what did we decide?" and get the motion, vote, mover and the exact second in the recording. Names are never guessed.

**Inspiration**
Volunteer boards (HOAs, nonprofits, school PTAs) run on memory and late minutes. When we tested an AI minutes pipeline on five real public meetings, it found 24 of 24 motions and outcomes but named the mover and seconder correctly only 16 and 14 times out of 24. Getting a name wrong in official minutes is worse than leaving it blank.

**What it does**
Board Clerk is a remote MCP server an Alexa+ style assistant can call. It ingests a meeting transcript, extracts every motion, outcome and tally, and marks a mover or seconder "spoken" only when that name was actually said near the motion. Anything else is flagged for the clerk to confirm with one tap, and only roster names are accepted. Minutes cannot be approved while names are unconfirmed. Every step is audit-logged, and nothing is emailed automatically. Members can ask "What did we decide about the roof?" or "Who seconded the pool resurfacing?" and get the motion, result, tally, mover/seconder and a "Listen at 00:22:10" timestamp. `prepare_board_packet` builds a topic packet with approved dollar totals, open confirmations and tabled items.

**How we built it**
Cloudflare Worker + MCP 2025-11-25 Streamable HTTP (@modelcontextprotocol/server), one Durable Object per board so memory persists across sessions. 16 tools with titles and annotations, resource templates, a `ui://board-clerk/decision-card` UI resource, an `ask_the_board` prompt, and an Agent Skill (skills/board-clerk/SKILL.md). The extractor is rule-based, costs $0 per meeting and the server makes no LLM calls. Guardrails: Origin allowlist, MCP only on /mcp, 256 KB cap, per-board caps, and a read-only shared demo board with isolated judge boards via `start_demo_board`.

**Challenges**
Attribution, not extraction, was the hard part, so we built the product around "never guess a name". Getting MCP routing right meant working around SDK behavior (see docs/FRICTION-LOG.md).

**Accomplishments**
Approval is blocked until a human confirms names. State carries across sessions. Every answer is a real tool call you can watch in the trace panel. 55 automated tests plus a remote smoke test.

**What we learned**
Assistants are only trustworthy for governance if they say when they don't know. Partial matches are labeled "Partial match" rather than hidden.

**What's next**
Real Alexa+ device testing, recording-to-transcript intake, and a video vote-count check (Roll Call Vision, our OpenCV entry).

**Built with:** cloudflare-workers, durable-objects, model-context-protocol, javascript, node.js, alexa

**Honesty notes (keep in text):** All demo data is fictional (Juniper Hollow HOA). The demo is a simulated Alexa+ experience and has not been tested on an Alexa device.

**Testing instructions:** paste docs/JUDGES.md (2-minute page path + Inspector CLI commands).
**Product feedback questions:** answer from docs/PRODUCT-FEEDBACK.md and docs/FRICTION-LOG.md.

## Remaining browser steps (Matt's accounts)
1. GitHub: push this repo (needs a GitHub login on the box or Matt's push). Matt decides: public MIT (qualifies for Open Source mini-prize) or private shared with judges.
2. YouTube: upload out/board-clerk-demo.mp4 as Public (or Unlisted if allowed), title "Board Clerk for Alexa+ — Hackathon Demo", attach captions.srt. Copy the URL.
3. Devpost (mbrady14): Amazon event -> Add project -> paste the fields above, demo URL, YouTube URL, repo URL, testing instructions, feedback answers -> Submit before Oct 23 1 PM MDT. Matt submits.
4. Optional LiftCheck edit: Matt republishes cbb7b4d on OpenAI Sites, then apply ../liftcheck-audit-1005/DEVPOST-UPDATE-DRAFT-1005-v3.md to entry 1200369.
5. Keep the worker live through judging (~Nov 20). Do not redeploy after the Oct 22 freeze.
