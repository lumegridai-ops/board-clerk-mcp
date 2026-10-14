# Board Clerk MCP: status (Sun Oct 11, 2026, ~9:45 PM MDT)
Entry: Amazon "Build, Ship, Shape", Alexa+ track (+ Open Source mini if public MIT repo). Deadline **Fri Oct 23, 1:00 PM MDT**; target submit Thu Oct 22, 3 PM.

## Live preview (synthetic data, new workers.dev name, not customer-facing)
- https://lumegrid-board-clerk-v02.lumegridai.workers.dev/demo (simulated Alexa+ page, real MCP client)
- https://lumegrid-board-clerk-v02.lumegridai.workers.dev/mcp (MCP 2025-11-25 Streamable HTTP)
- Version 0.3.0 = git 11f4c30, Cloudflare version 4980205c-b685-49d7-8d10-c1459d437d7c (deployed ~9:40 PM MDT Oct 11). Health on the v02 host still said 0.3.0 with 15 tools at 2026-10-12 03:49 UTC. Red-team edits and round 4 (read-only shared board, 16 tools, packet, decision card, skill, wrangler name `lumegrid-board-clerk-v02`) are local only and were not deployed. A future deploy of this file would publish onto the judge worker `lumegrid-board-clerk-v02`, not the old `lumegrid-board-clerk-mcp` host. Do not deploy from this task.
- Rollback: `npx wrangler rollback 817a87fc-3a7f-4fc3-a606-0960c92f28db --name lumegrid-board-clerk-v02` (v0.2.1); earlier v0.2.0 = ed06d8f4-5b9a-47d1-92cb-bfd99a268d4f.
- Old `lumegrid-board-clerk-mcp` worker (v0.1.0) untouched.

## Done
- Engine library (validate / crosscheck / spoken-name rule / $0 rule-based extractor / query / packet / minutes); Durable Object state per board; 16 tools with titles + annotations, including `prepare_board_packet`; resource templates plus `ui://board-clerk/decision-card`; `ask_the_board` prompt; Agent Skill `skills/board-clerk/SKILL.md`; approval guardrail; audit log; isolated judge boards (`start_demo_board`); `juniper-hollow-demo` read-only for writes; Origin allowlist, MCP only on /mcp, 256 KB cap, per-board caps. Crosscheck runs only when ingest is given `second_pass_motions`. The server does not call Gemini.
- Tests: 55/55 `npm test` after round 4, plus engine and MCP-local smokes. Earlier remote smoke vs preview and the MCP Inspector CLI cross-session flow were Oct 11, before this local round.
- Docs: README, docs/JUDGES.md, docs/ARCHITECTURE.md, docs/FRICTION-LOG.md, docs/PRODUCT-FEEDBACK.md (friction item (a) re-verified by Nova: SDK 2.3.1 `createMcpHandler({route:"/mcp"})` answers initialize on `/` and `/other` with 200), docs/REVIEW-FINDINGS-1011.md. CI workflow (.github/workflows/ci.yml), MIT LICENSE.
- Demo narration rendered: 8 scenes, gemini-3.8-flash-tts (voice Charon), 164 s, ~4.6k tokens total, est. < $0.10 (ledger `../board-clerk-video/tts-ledger.jsonl`; one 429 on S6, no audio/charge, re-sent after a pause). Video build in `../board-clerk-video/`.

## Left (owner)
1. Demo video final cut + Pika QA (Nova build, Pika QA).
2. Red-team pass on full package + Pika QA of preview (Nova, Pika).
3. Repo hosting: GitHub push needs GitHub login on box (Matt). Public MIT (enables Open Source mini) vs private shared with Amazon judges (Matt decision).
4. Devpost project under mbrady14 + paste text, video URL (YouTube upload, public), feedback answers (Matt creates project; Pika/Nova fill after QA agreement).
5. Freeze on Oct 22; keep preview live through Nov 20 judging.

## QA escalation rule (Matt, Sun Oct 11 10:07 PM MDT): nothing waits on non-response
1. QA ask goes to @pika via team-chat/post.sh (one post per milestone).
2. No Pika verdict within 1 h -> ask @juno.
3. No Juno answer within another 1 h -> rigorous self-review: separate Grok Build xhigh red-team run, findings fixed, then proceed.
4. Still must be top-tier: if the self-review says not top-tier, keep improving. Never submit; Matt creates/submits on Devpost.
Clock log (MDT):

## Round 4 deploy (Sun Oct 11 ~10:25 PM MDT), commit d2ec60c
- lumegrid-board-clerk-v02 -> version a75b30b0-e47d-4030-b8f2-4d3709e9d0de. Rollback: `npx wrangler rollback 4980205c` (prior v0.3.0).
- lumegrid-board-clerk-mcp (old v0.1 name, which said "do not submit") redeployed with the same code -> 7991ade8-41f7-4972-98e4-dcdd09c28c88. Its "do not submit" text is gone.
- Remote smoke passed on both; Inspector tools/list = 16; read-only default board verified.
- Video re-captured from the live round-4 page: out/board-clerk-demo.mp4 166.1 s, 1080p30, -16 LUFS, qa.sh PASS. Narration unchanged (no new TTS spend). Stat card now says "Our earlier AI-minutes pipeline test".
- Version bumped to 0.4.0 and redeployed: v02 -> 4425ff17-925a-4f14-8ad8-68483fcc9afa, old name -> 9bcf5d6c-b500-4dd1-8d70-f0a224d97dbe (55/55 tests, remote smoke passed on both). Rollback v02: `npx wrangler rollback a75b30b0-e47d-4030-b8f2-4d3709e9d0de` (round 4 at 0.3.0 label) or 4980205c (v0.3.0).
- Sun Oct 11 22:35 MDT: QA ask posted #2018 to @pika (Board Clerk v0.4.0 package + video; GitLab scope). Escalate to @juno at ~23:35 MDT if no verdict; Grok xhigh self-review at ~00:35 MDT if Juno silent.
- Sun Oct 11 23:48 MDT: Pika hour elapsed with no Board Clerk verdict. Escalated to @juno via team-chat #2101. Nova Grok xhigh self-review ~00:35 MDT if Juno silent.

- Sun Oct 11 23:52 MDT: Juno backup QA **PASS** (#2105) on live v0.4.0 package (root/demo/mcp tools/list). Pika independent agree still required before any Devpost submit. No submit; no card/spend.
- Mon Oct 12 00:51 MDT: Overnight ladder — Board Clerk closed on Juno #2105; no self-review needed.
- Mon Oct 12 01:52 MDT: Overnight ladder — no new escalations. Board Clerk/DisputeKit/captions/tribute closed prior; batch-1012 + funeral Cap ops HOLD; batch-1011 waits Juno 8:30 AM; batch-1013 Juno HOLD #2144 (Matt 11:47) — no self-review PASS / no send-ready. Room posts: 0.
- Mon Oct 12 03:42 MDT: Overnight ladder — no new escalations. Board Clerk/DisputeKit/captions/tribute closed prior; batch-1012 + funeral Cap ops HOLD #2119; batch-1011 waits Juno 8:30 AM; batch-1013 Juno HOLD #2144 (Matt 11:47) — no self-review PASS / no send-ready. Room posts: 0.
- Mon Oct 12 04:41 MDT: Overnight ladder — no new escalations; room quiet since #2163. Same holds as 03:42 (batch-1012/funeral Cap ops HOLD #2119; batch-1011 waits Juno 8:30 AM; batch-1013 Juno HOLD #2144). Room posts: 0.
- Mon Oct 12 05:50 MDT: Overnight ladder final run — no new escalations; room #2164–#2166 is Ep012 release only. Same holds (batch-1012/funeral Cap ops HOLD #2119; batch-1011 waits Juno 8:30 AM; batch-1013 Juno HOLD #2144). Room posts: 0. Ladder routine retired.
