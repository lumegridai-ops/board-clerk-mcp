# Red team, 2026-10-11 (probe 2026-10-12 03:49 UTC)

Hostile read for Amazon "Build, Ship, Shape", Alexa+ track. Official pages fetched that day: [landing](https://amazonappdev2026.devpost.com/), [rules](https://amazonappdev2026.devpost.com/rules), [resources](https://amazonappdev2026.devpost.com/resources). Deadline on those pages: Oct 23, 2026, 12:00pm PDT. Alexa+ prizes listed there: $25,000 / $15,000 / $4,000 plus AWS credits. Judging window in the rules: Nov 9–20, 2026.

This pass did not deploy, did not commit, and did not call any tool on `juniper-hollow-demo`. The live check used one fresh board from `start_demo_board`, then `reset_board` on that same id (`rt1011-72e5d56e`). `GET /demo` was fetched as HTML only. The page's JavaScript was not executed in a browser, so the page itself did not create a board.

## What the live host actually did

`GET https://lumegrid-board-clerk-v02.lumegridai.workers.dev/` and `GET /health` returned `board-clerk` `0.3.0`, protocol `2025-11-25`, transport `streamable-http`, 15 tools. `POST /mcp` `initialize` with `2025-11-25` returned `protocolVersion: "2025-11-25"`, `serverInfo.version: "0.3.0"`, `content-type: text/event-stream`, no `mcp-session-id`. `GET /mcp` and `DELETE /mcp` returned 405 `Method not allowed.` That is the SDK's stateless legacy path, not a sessionful stream. State survived across later POSTs with no session header, which means the Durable Object, not the HTTP session.

`tools/list` had titles and `readOnlyHint` / `destructiveHint` / `idempotentHint` / `openWorldHint: false` on all 15 tools. `prompts/list` returned `ask_the_board`. Resource templates were `board://{board_id}/roster` and `board://{board_id}/meetings`.

On the fresh board:

- Roof question spoke the August 20 Summit Ridge motion, "forty-seven thousand nine hundred", 4 to 1, Priya Raman moved, Luis Trevino seconded.
- "Who seconded the reserve study?" spoke the September 17 Blue Fern pool motion as the answer. It did not say there is no reserve-study vote. Top hit was `juniper-2026-09-17` M3.
- Six fields needed confirmation. Confirming August 20 M3 mover `Okafor` and seconder `Lindqvist` stored `Dana Okafor` and `Grace Lindqvist`. A later POST on the same board showed August pending count 0.
- Approving September with August's approval id returned `isError` (id does not match the meeting). Approving August returned `approved` and "No email was sent."
- `Not A Person` was rejected with the roster.
- `Origin: null` and `Origin: https://localhost` were 403 `{"error":"origin not allowed"}` and did not echo the header. `Origin: http://localhost:8787` was allowed. `POST /`, `POST /mcp/`, and `POST /other` were 404.
- `GET /demo` was HTML, CSP set, banner "Simulated Alexa+ experience", no `speechSynthesis`, version string `0.3.0`. `GET /demo/transcript.txt` was the fictional October 15 transcript.

A different hostname, `https://lumegrid-board-clerk-mcp.lumegridai.workers.dev/`, answered `GET /` as version `0.1.0` with an `amazon_status` that called the build a scaffold and said Board Clerk should stay unsubmitted because LiftCheck had Alexa+ priority. `wrangler.toml` in this tree still names that worker. `npx wrangler deploy` from this checkout would publish there, not to the v02 judge URL. That old sentence is public.

## Stage 1

Rules, section 6: Stage 1 is pass/fail on whether the project reasonably fits the theme and reasonably uses the required APIs. It is not the four scored criteria. Those are Stage 2, equally weighted. A submission can clear Stage 1 and still lose on "basic MCP wrapper."

| Requirement | Verdict | Why |
|---|---|---|
| Alexa+ required tech: self-hosted MCP, spec 2025-11-25 or later, Streamable HTTP, or an Agent Skill, or a simulated web experience | Pass for the project | Live initialize negotiated `2025-11-25` over HTTP and returned SSE. `/demo` is a simulated page that calls `/mcp`. No Agent Skill file. The rules use "or", so the missing skill is not a Stage 1 fail. |
| Repo actually calls the track tech, not a README mention | Pass | `src/server.js` registers the tools and `src/index.js` serves `/mcp`. Confirmed live, not only by reading. |
| Working demo a judge can open through Nov 20 | Pass if the worker stays up | No login. Isolated boards work. The default id `juniper-hollow-demo` is world-writable. A stranger can `reset_board` it before a judge opens the shared id. |
| Code repository with source, assets, and run instructions; public with a detectable OSI license, or private and shared with the listed Amazon GitHub accounts | Not in this tree | MIT `LICENSE` is present. This pass did not prove a GitHub remote, collaborator invites, or that invitations are unexpired. Rules: invites die after 7 days. |
| Demo video under 3 minutes, public YouTube or Vimeo, English, product shown working | Fail as a submission | No video is in this repo. `STATUS.md` still lists the cut as not done. Judges may stop at 3:00. |
| Product feedback for each tool, API, or SDK, on the form | Draft only | `docs/PRODUCT-FEEDBACK.md` exists. It is not a Devpost answer. Two items were inaccurate until this pass (below). |
| Optional friction log (up to 10% of the Stage 2 score, recommended at Stage 1) | Present, bonus not safe at 10% | Items (e) and (f) have no saved log. Items (a) and (b) misstated the SDK until this pass. A reviewer who opens `@modelcontextprotocol/server` 2.3.1 can deny the bonus. |
| English materials | Pass | |
| Original work, substantially different from the entrant's other submissions | Risk, not a code fail | See LiftCheck. Organizer discretion. Rules section "Multiple Submissions." |
| One track prize and one mini-challenge prize | Do not also enter AWS Builder | This server does not call Bedrock, AgentCore, Strands, Kiro, or SageMaker. Cloudflare Workers is not that mini. Open Source mini wants an *additional* public contribution during the window, not "the primary repo is MIT." |
| Fits the theme | Pass | A clerk loop is a real Alexa+ shaped job. It is not Fire TV, Bee, or Ring. |

**Stage 1 call:** the software clears the viability bar. The package you would upload today does not. Missing video, no proof the judges can see a GitHub repo, and a second Alexa+ entry beside LiftCheck are the disqualifiers. The public `0.1.0` worker telling the world not to submit this project is an own goal if anyone finds that hostname.

## Stage 2 scores

Scale is 1–10. The rules do not publish a point scale. These are this reader's marks, equal weight. Higher is better. Live behavior is what was probed. Local edits below are not on the worker.

| Criterion | Score | Reason |
|---|---|---|
| Tech implementation | 7 | Real MCP server, not a README diagram. Protocol pin matches the minimum. Tools, annotations, resources, prompt, caps, origin allowlist, and the approval guardrail all ran. Tests cover the guardrail. Docks: stateless 405 on `GET /mcp` (acceptable for this SDK's legacy mode, easy to misread as "not Streamable HTTP"); `board_id` is the only capability; `src/engine/crosscheck.js` is not on the ingest path; `scripts/smoke-mcp-local.mjs` used to boot a second server that called itself `0.1.0` (fixed locally); wrangler name points at the wrong worker. |
| Design | 6 | `/demo` is a coherent simulated display: banner, chips, decision card, confirm buttons, minutes refusal, MCP trace. It does not speak. Docks: the live reserve-study chip answers a different motion in a confident voice; "Draft last night's minutes" drafts September 17 until October 15 is ingested; the router is a regex, not an agent; no MCP App `ui://` card. Local tree now says when the match is partial and when the draft is not October 15. The public page does not. |
| Potential impact | 6 | The job is specific: community-association staff who have to name the mover and seconder before minutes can be approved. That is a real audience beyond the weekend. Docks: every record is synthetic; no manager used it; the old README accuracy line was a different system's numbers; nothing here sends minutes anywhere a board would actually read them. Credible story, no evidence. |
| Quality of the idea | 5 | Not a single-turn joke, and not a thin client of an HOA vendor API. The creative piece the rules list is "context-aware add-on that maintains state across sessions," and the Durable Object does that. The surface a judge hits first is still keyword overlap plus a Q&A chip. The rules' creative list also names purchasing, cards and carousels, MCP Apps, and Agent Skills. None of those are here. `ask_the_board` used to tell the model to call one search tool. It now describes confirm-then-approve. That text is local only. |

Total 24/40. Friction bonus, if believed after the corrections: low single digits, not 10%. Tie-break order in the rules is the criterion order above, so Tech is the first tie-break. This entry does not win a tie on idea.

## "Basic MCP wrapper" risk

The rules, Quality of the Idea, Alexa+: obvious is "single-turn Q&A bot, basic MCP wrapper around an existing API." Creative is an agentic workflow across services, state across sessions, purchasing, media cards, MCP Apps, Agent Skills.

This is not a wrapper around someone else's HTTP API. There is no upstream board API. The engine, the spoken-name window, and the "do not approve while a name is unconfirmed" rule are the product. A judge who only runs the roof chip will still call it a wrapper: one question in, one paragraph out, keyword score, no tool loop unless the human clicks again. The live reserve-study answer is the exhibit. It treats "reserves" as "reserve study" and speaks the pool vote as the decision.

What keeps it off the bottom of that scale:

- Confirmations and approvals are still there on a new HTTP call with the same `board_id` and no session cookie. Probed.
- `approve_minutes` fails closed. Probed for a mismatched approval id; the pending-name refusal is in `test/mcp.test.mjs`.
- The demo shows the tool trace, so it is not a chatbot pretending it did not use tools.

What a hostile reader will say anyway: fifteen tools that read and write one SQLite object, search by token overlap, and a web page that `fetch`es `/mcp`. That is the shape of a weekend MCP wrapper with a domain noun. State across sessions is the one creative checkbox you can defend without blushing. Lead the video with the refusal, then the confirm, then the second session, not with "what did we decide about the roof?"

## Uniqueness versus LiftCheck

LiftCheck is the same team's other Alexa+ entry: transit elevator path checks over MCP (Devpost software/liftcheck, submission 1200369 in the team's notes). Public story: station entrance to platform, current alerts, alternate elevators, spoken limit that a clear report is not a promise the elevator works. Hosted MCP, protocol 2025-11-25, plus a simulated page. Their later notes add `voice.text`, two prompts, an Agent Skill file, and a bearer workspace key so a cookieless client can find a saved journey.

Board Clerk is a different user and a different job. A rider asking whether an elevator path is open is not a clerk asking who seconded a roof bid, blocking approval until a human confirms the name, and writing an audit row. Data differs: LiftCheck uses a real transit feed and labeled synthetic outages. Board Clerk uses only fictional meetings and does not call a model. The shared parts are the ones a skeptical organizer will stare at: same track, same season, MCP 2025-11-25 on a hosted URL, a simulated Alexa page, cross-session memory, no tested Alexa device, and a friction note about docs. The rules say each submission must be unique and substantially different, in the sponsor's sole discretion. Same Devpost account, two Alexa+ projects, is how entries get merged or one of them gets dropped.

On the merits this is different enough to argue. It is not different enough to assume. The `0.1.0` health string that names LiftCheck and tells readers to hold this project back will be read as the team agreeing with the drop. Take that string off the public internet before anyone pastes hostnames into a form. That is an infra change. This pass did not do it.

LiftCheck is also ahead on the creative list (Agent Skill file, spoken caveat baked into every result). Board Clerk is ahead on a fail-closed write and on per-board durable state. Do not describe them as one platform with two skins.

## Five reasons a judge would pass over this, and the fix

1. **Second Alexa+ entry, plus a public worker that withdraws it.** Fix: one sentence at the top of the Devpost text on how this is not LiftCheck (governance minutes and an approval gate, not elevator paths). Remove or replace the `0.1.0` worker's withdrawal health JSON. Do not deploy this tree onto that name by accident. Not done here, because that is the live worker.

2. **The first chip sounds like single-turn search, and the second chip answers the wrong vote.** Fix: `what_did_we_decide` now says when a topic word is missing, then gives the closest match in the same two sentences (`src/engine/query.js`). The roof answer no longer glues "passed" onto the last clause of the motion. `ask_the_board` now tells the client to confirm names and not to declare approval when the tool returns an error. **Not deployed.** The host probed at 03:49 UTC still speaks the pool motion as the reserve-study answer.

3. **Borrowed accuracy.** The README used to say a MinutesDesk test of 5 public meetings was 24/24 motions and outcomes, movers 16/24, seconders 14/24, and that the production path is Gemini two-pass plus crosscheck. Nothing in this repo computes those ratios. `crosscheck.js` is not called from ingest. This server does not call Gemini. An internal note outside this repo also says that run was not accepted as checked against the recordings. Fix: `README.md` Accuracy now says the window is unit-tested on synthetic lines and that an outside model test is not this server's score.

4. **The upload is incomplete.** No video, no shown GitHub share with the six Amazon accounts, feedback still a markdown file. Fix is submission work, not another tool. Film the refusal and the second session in the first 45 seconds. Paste the feedback answers, including the corrected SDK notes, into the form. If you enter Open Source, point at an additional contribution made during the window, not only this MIT file.

5. **The shared board is the default, and it is writable.** Omit `board_id` and you are on `juniper-hollow-demo`. `reset_board` and `ingest_meeting` work there, except ingest will not overwrite the two fixture ids. A judge, or anyone else, can layer junk onto the board the next judge opens. Fix, not applied: make destructive tools require an explicit id, or make that one id read-only and keep writes on `start_demo_board` copies. The current tests and README define the default as that shared id, so this pass left the contract alone. Tell judges to use `?reset=1` or their own id, and assume the shared id will get dirty.

## Claims that were not supported

Line numbers are from the files as read at the start of this pass, before the rewrites in the same pass.

| Claim | Where | Why it was not supported | Now |
|---|---|---|---|
| Public host health was `0.2.0` on 2026-10-11, and this `0.3.0` checkout was not deployed | `README.md:20`, `docs/JUDGES.md:7` | `GET /` at 2026-10-12 03:49 UTC returned `0.3.0` with the 15-tool list. `STATUS.md` already said a 0.3.0 deploy happened ~9:40pm MDT Oct 11. | Rewritten. Current text: `README.md:22`, `docs/JUDGES.md:7`. Later local edits are still not deployed. |
| "On the MinutesDesk test of 5 public meetings, motions and outcomes were 24/24. Movers were 16/24 and seconders were 14/24." And "The production path is a Gemini two-pass extract plus crosscheck." | `README.md:82` | No script, fixture, or test in this repo produces 24/24, 16/24, or 14/24. Ingest calls `extractMeeting` only. `crosscheck.js` is not imported by `src/server.js`. The server does not call Gemini. The next sentence did say the server does not call Gemini, which made the "production path" sentence read as this product. | Removed from the README. Accuracy is `README.md:84-90`. |
| `@modelcontextprotocol/server` 2.3.1 `createMcpHandler` did not enforce its `route` option | `docs/FRICTION-LOG.md:5-13`, `docs/PRODUCT-FEEDBACK.md:7` | Installed 2.3.1 `CreateMcpHandlerOptions` has no `route` field. Passing `route: "/mcp"` did nothing because the option does not exist. The extra paths were real: the worker forwarded them. The cause was misstated. | Corrected in those files and in `docs/REVIEW-FINDINGS-1011.md` H2. `createHandler` no longer passes `route`. |
| The handler has no Origin validation, and what would have helped is Origin validation in the SDK | `docs/FRICTION-LOG.md:15-23`, `docs/PRODUCT-FEEDBACK.md:7-9` | `createMcpHandler` is validation-free on purpose. The same package exports `originValidationResponse`, `validateOriginHeader`, and `localhostAllowedOrigins`. The transport can check `allowedOrigins`. The custom allowlist and the DNS-rebinding hole in the first version are real. "The SDK does not have this" was not. | Corrected. The workers.dev exception is still ours. |
| The device-docs gap is the same pain as the LiftCheck feedback | `docs/FRICTION-LOG.md:39`, `docs/PRODUCT-FEEDBACK.md:31` | LiftCheck's written note cites Amazon account-linking and authentication pages and says those pages exist. This note is "we found no attach path for a remote MCP during the event." Those are different claims. | Softened. The "no click-path we found" observation is still only one look, as the log says. |
| Inspector CLI completed `tools/call` on the remote host, including a JSON array roster argument | `docs/FRICTION-LOG.md:49`, `docs/PRODUCT-FEEDBACK.md:13`, `docs/JUDGES.md:84` | No Inspector transcript, command log, or saved output is in this repo. This pass did not run the Inspector CLI. A `fetch` client did complete the tool sequence on a fresh board. It did not send a roster array. | Labeled as an unreproduced note. The fetch run is recorded in `docs/JUDGES.md:84`. |
| Wrangler 3.114 printed an upgrade-to-4 warning, and headless OAuth did not survive a restore | `docs/FRICTION-LOG.md:59`, `docs/PRODUCT-FEEDBACK.md:25` | No warning text or login log is in the repo. Not disproved. Not evidenced. | Labeled as an unreproduced note. |

Checked against code or the live host and left standing: protocol `2025-11-25`; 15 tools with titles and annotations; resources and `ask_the_board`; per-board Durable Object with `MemoryStore` for tests; fixture layering only on `juniper-hollow-demo`; `start_demo_board` refuses that id; spoken-name window −45s to +30s (`src/engine/namerule.js`, `test/namerule.test.mjs`); approval `isError` while a name is unconfirmed (unit test; live check covered the mismatched-id error and a successful approve after confirms); 200 KB / 25 meetings / 40 approvals / 200 audit / 80-character caps in `src/state.js`; origin allowlist behavior probed above; demo page does not call `speechSynthesis`; no model and no email call in `src/`. `npm test` after the edits: 48 passed, 0 failed. `npm run test:engine` and `npm run test:mcp-local` passed. `test:mcp-local` now drives `createHandler` from `src/server.js` and expects `0.3.0`. It used to construct a separate 8-tool server at version `0.1.0`.

## Local fixes in this pass (not deployed)

- Spoken answer: outcome is not glued to the last clause; a missing topic word is named; closest match stays inside two sentences.
- Repeating `approve_minutes` returns the stored approver name instead of the new argument.
- `draft_minutes` includes `meeting_date`. The demo page says when "last night" is not the October 15 transcript.
- `ask_the_board` describes the confirm and approve loop and forbids inventing an approval.
- Dead `route` option removed from `createHandler`.
- `scripts/smoke-mcp-local.mjs` calls the real handler.
- README, JUDGES, FRICTION-LOG, PRODUCT-FEEDBACK, ARCHITECTURE, REVIEW-FINDINGS H2, and STATUS updated so they do not repeat the claims above.

`npm test` was run after those edits: 48/48. Nothing was committed. The v02 worker and the old `lumegrid-board-clerk-mcp` worker were not modified.

## Round 4 response

Local only. Nothing was deployed, and neither live worker was changed. The probe above still describes the host at 2026-10-12 03:49 UTC (version `0.3.0`, 15 tools). This section maps that probe to the tree after round 4.

| Finding | Status |
|---|---|
| `wrangler.toml` named `lumegrid-board-clerk-mcp`, so a deploy from this tree would miss the judge URL | Fixed locally. `name` is `lumegrid-board-clerk-v02`. A future deploy would hit the judge worker. Not deployed. The old `0.1.0` host was not modified, so its withdrawal health JSON is still public. |
| Public `0.1.0` health JSON withdraws the project and names the other entry | Not fixed on the internet. The exact withdrawal phrases were removed from `src/` and `docs/` in this tree. Taking the sentence off the live worker is an infra change this pass did not do. |
| Second Alexa+ entry beside LiftCheck | Not fixed. Still a Devpost sentence (governance minutes and an approval gate, not elevator paths), not a code change. |
| Shared board `juniper-hollow-demo` is world-writable (`reset_board`, `ingest_meeting`) | Fixed locally. `ingest_meeting`, `confirm_mover_seconder`, `request_minutes_approval`, `approve_minutes`, and `reset_board` return `isError` on that id and tell the caller to use `start_demo_board`. Reads still layer the fixtures and set `demo_board: true`. The demo page already uses its own board. Not deployed. |
| `crosscheck.js` is not on the ingest path | Fixed locally when the caller passes `second_pass_motions`. Those motions are checked with `validate.js` before anything is stored. A mover or seconder disagreement becomes `NEEDS CONFIRMATION`. The server still does not call Gemini. Omitting the second pass does not invent one. |
| No Agent Skill file | Fixed locally. `skills/board-clerk/SKILL.md` (name, description, then the clerk loop). README and `docs/JUDGES.md` link it. Not a substitute for the missing video or the GitHub share. |
| No MCP App `ui://` card | Implemented locally with a plain resource and tool `_meta`, not with `registerAppTool`. URI `ui://board-clerk/decision-card`, mime type `text/html;profile=mcp-app`, link `_meta.ui.resourceUri` on `get_decision` and `what_did_we_decide`, resource `_meta.ui.prefersBorder`. Text and `structuredContent` remain. Installed `@modelcontextprotocol/server` 2.3.1 does not export `registerAppTool` or `registerAppResource` and this server does not advertise `io.modelcontextprotocol/ui`. No MCP Apps host was run, so iframe rendering is not verified. In-process tests check the resource bytes and the tool link. |
| Quality of the idea: no purchasing, cards, MCP Apps, or Agent Skills; surface is keyword Q&A | Partially fixed locally. The skill, the decision-card resource, and `prepare_board_packet` (topic across meetings, approved dollar total, open confirmations, tabled items, no model) are in this tree. Purchasing is not in this product. The demo router is still a regex. Not deployed, so the host a judge opens does not have them. |
| Reserve-study chip answers a different motion; live page does not say the match is partial | The chip in this tree is now "Who seconded the pool resurfacing?" (September 17, Blue Fern, Mark Ellison seconded). "Board packet: roof" calls `prepare_board_packet`. A partial topic still says so on the page ("Partial match") and in the spoken line. The public page checked at 03:49 UTC still has the old chip. Not deployed. |
| Borrowed accuracy (24/24, movers 16/24, seconders 14/24, and a Gemini production path read as this server) | README Accuracy now says those ratios come from a separate Gemini MinutesDesk pipeline test on 5 public meetings, not from this rule-based extractor. This repo does not compute them and does not call Gemini. An outside note did not accept that run as checked against the recordings, so the README does not call them recording-verified. The extractor's stated scope is clean synthetic `[HH:MM:SS] Speaker:` transcripts. The −45s to +30s window stays a unit test on synthetic lines. |
| Upload incomplete: no video, no shown GitHub share, feedback still a markdown file | Not fixed. Still submission work. |
| Stateless `405` on `GET /mcp`; `board_id` is the only capability | Not fixed. Same as the probe. `GET /mcp` stays the SDK legacy 405. L1 in the review findings stays open. |
| Friction log items (a)–(f) | Left as written. Those SDK and tooling notes did not change in round 4. |

After these edits, `npm test` passed 55/55, and `npm run test:engine` and `npm run test:mcp-local` passed. Nothing was committed. The v02 worker and the old `lumegrid-board-clerk-mcp` worker were not modified.
