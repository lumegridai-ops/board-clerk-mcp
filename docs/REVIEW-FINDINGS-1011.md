# Review findings (2026-10-11)

Independent pass over `src/` as a hackathon judge and a security reviewer, after the round-2 behavior was in place. Line numbers are from that tree. High and Medium items below are fixed. Nothing was deployed.

## High

| # | Finding | Where | Fixed? |
|---|---------|--------|--------|
| H1 | Origin allowlist treated "same as the request URL" as the worker's own origin. A DNS-rebind page whose name resolves to local `wrangler dev` sends `Origin: http://evil.example` to `http://evil.example/mcp`, so the two match and the check would have allowed the call. | `src/index.js:18-40` | Yes. `http://localhost` and `http://127.0.0.1` (any port) are allowed. Same-origin is allowed only when the host is `*.workers.dev`. `https://localhost` and `Origin: null` are rejected. The 403 body is `{error:"origin not allowed"}` and does not echo `Origin`. Missing `Origin` stays allowed. |
| H2 | `@modelcontextprotocol/server` 2.3.1 `createMcpHandler` has no `route` option, so a `route: "/mcp"` argument does nothing and the handler serves MCP on every path it is mounted on. Origin and the 256 KB cap ran only for `pathname === "/mcp"`. `POST /` and `POST /mcp/` skipped both and still executed tools. | `src/index.js` path gate | Yes. After `GET /`, `GET /health`, `GET /demo`, and `GET /demo/transcript.txt`, any other path is 404. `/mcp` always runs the origin check, then the body cap (Content-Length, then a capped stream read). The dead `route` argument was removed from `createHandler`. |

## Medium

| # | Finding | Where | Fixed? |
|---|---------|--------|--------|
| M1 | The public endpoint has no auth. `request_minutes_approval` minted a new id on every call, and every write appended an audit row. `approver_name` and `confirmed_by` were unbounded strings. A client could grow Durable Object storage without a limit. | Caps in `src/state.js:10-15`, `src/state.js:106-116`, `src/state.js:130-136`. Name checks in `src/server.js:411-416`, `src/server.js:553-557`, `src/server.js:632-636`. | Yes. 40 approval requests and 200 audit entries per board, then an error that says to call `reset_board`. Both name fields are capped at 80 characters and return `isError`. |

## Low (left as specified)

| # | Finding | Where | Fixed? |
|---|---------|--------|--------|
| L1 | `board_id` is the only capability. Anyone who knows it can read and write that board. Isolated demo ids are `judge-` plus 12 hex chars. | `src/server.js:82-86`, `src/state.js:16-21`, `src/ui/demo-html.js:229-235` | No. This is a public demo, not a multi-tenant product. |
| L2 | `juniper-hollow-demo` still accepts an ingest whose id does not collide with a fixture, until someone calls `reset_board`. Collision with `juniper-2026-08-20` or `juniper-2026-09-17` is refused. | `src/server.js` write handlers (`sharedWriteRefusal`) | Yes, in round 4, local only. The shared board was specified as writable. Every write tool now returns `isError` on that id and tells the caller to use `start_demo_board`. Reads still layer the fixtures. Not deployed. |
| L3 | The demo CSP allows `style-src` and `script-src` `'unsafe-inline'` because the page is one file with no build. User text is assigned with `textContent` only. | `src/index.js:100-101`, `src/ui/demo-html.js:163-170` | No. |
| L4 | `http://[::1]` is not an allowed Origin. The required allowlist is `localhost` and `127.0.0.1`. | `src/index.js:14-15` | No. |
| L5 | There is no separate Host-header allowlist. Browser rebinding is covered by H1. Clients that omit `Origin` are allowed on purpose. | `src/index.js:18-21` | No. |

## Round-2 behavior checked

These were verified in code and by `npm test`. Not regressions.

| Item | Where | Fixed? |
|------|--------|--------|
| Agenda from chair cues; org and title from args or a `#` header; meeting id slug from org + date | `src/engine/extract.js:107`, `src/engine/extract.js:151`, `src/server.js:158-174`, `src/server.js:486-494` | Yes |
| Absent roll-call members are not voice-vote ayes | `src/engine/extract.js:265-274`, `src/engine/extract.js:320-335` | Yes |
| MCP confirm matches a roster member and stores the full name. Engine `confirmMoverSeconder` still accepts a raw value such as `Ellison`. | `src/server.js:188-223`, `src/engine/query.js:249-267` | Yes |
| Default board reports `demo_board: true` on reads. Round 4 refuses `reset_board` and the other writes on `juniper-hollow-demo` instead of restoring fixtures by wiping that id. `start_demo_board` copies the two fixtures and rejects `juniper-hollow-demo`. | `src/server.js` `sharedWriteRefusal` and `start_demo_board` | Yes |
| Tool errors are `isError` results with a next step (unknown meeting, pending confirms, limits, fixture shadow) | `src/server.js:78-80` and the tool handlers | Yes |
| `top_answer` is two sentences, natural date, amount as written, unconfirmed names called out | `src/engine/query.js:307-356` | Yes |
| `GET /demo` is one HTML file, no CDN, MCP `2025-11-25`, no `speechSynthesis` | `src/ui/demo-html.js`, `src/index.js:94-104` | Yes |

Checked and not exploited: fixtures are `structuredClone`d before copy; approval text says no email was sent; the worker does not call a model.
