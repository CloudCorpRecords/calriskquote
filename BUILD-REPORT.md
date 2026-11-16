# BUILD REPORT — calriskquote

Clean-room rebuild of `com.douxprocess/calrisk-insurance-center`
v1.0.0 (remote-only MCP server at
`https://douxprocess.com/mcp/a/calrisk-insurance-center`), built
2026-10-28 as a forwarding client. Package: `@reneturcios/calriskquote`
0.1.0, registry name `io.github.CloudCorpRecords/calriskquote`.

## Surface built

Labelled subset: **3 of 4 captured tools built**, 1 omitted (see below).

| Tool | Required fields as served |
|------|---------------------------|
| `agency_info` | NO `required` key in the capture — served verbatim, none invented |
| `check_appetite` | `line`, `state` |
| `start_online_quote` | NO `required` key in the capture — served verbatim, none invented |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured (no
`additionalProperties` and no `$schema` marker on any inputSchema,
matching the fixture). The tool literals were generated
programmatically from
`harness/fixtures/douxcalrisk-tools-original.json` (THIS agency's
fixture, not any sister agency's) by the script at
`/tmp/b44-calriskquote-scratch/gen.py` (parse + re-serialise into
`src/forward.ts`) — never retyped — and a post-generation check (the
registration-integrity test suite, plus the parity harness) confirmed
the served literals deep-equal the fixture's 3 tool objects, in
order. Tools are registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers), not
McpServer+zod.

## Omission — `request_quote`

- OMITTED: `request_quote` (required: `name`, `line`, `state`,
  `details`, `consent`) — SENDS a person's name, an email or phone,
  the line, the state and free-text coverage details to the agency's
  licensed staff for a reply: a send-shaped PII lead-routing tool
  (the porefind / spinspec omission family), omitted per the spec.
  The consent flag does not cure the shape.
- It is NOT registered, does NOT appear in tools/list, and is NOT a
  CLI command.
- Absence assertions in `test/forward.test.ts`:
  - Registry level: `TOOL_NAMES` does not include it, no `TOOL_DEFS`
    entry carries its name, `TOOL_DEFS.length === 3`, and
    `isKnownTool("request_quote") === false`.
  - MCP layer: `handleCallTool` rejects it with McpError
    MethodNotFound before any network hop (upstream call count 0).
  - CLI level: `calriskquote request_quote '{}'` prints Usage on
    stderr and exits 2, and the usage tool list does not name it.

## Fixture conflicts

**None.** The spec and the fixture agree on the built tools' names,
titles, descriptions, required fields (none for `agency_info` and
`start_online_quote`; `line` + `state` for `check_appetite`) and
property shapes, including the six-line `line` enum
(`general-liability`, `workers-comp`, `commercial-auto`, `bonds`,
`property`, `professional`) carried by both `check_appetite` and
`start_online_quote`.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned:
  `https://douxprocess.com/mcp/a/calrisk-insurance-center`.
- Env override `DOUXCALRISK_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header is
  sent under any environment — including with key-like env vars
  (`DOUXCALRISK_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and that
  no `X-API-Key` header is ever sent; upstream requests carry only
  `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `calriskquote error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/calriskquote`.
- Package entries: **128**.
- sha256: `d0ab12a3ca2be2eb51b0f4587a32dbe9d73bf253bba1c955b34c26541cd84a1c`
  (re-verified unchanged after the subsequent offline `npm install`).

## server.json

- Description exactly: "CalRisk Insurance Center: agency profile,
  appetite checks and online quote links for CA lines." — **94
  characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/calriskquote:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/calriskquote"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/calriskquote`
  and its smoke test asserts the server name `calriskquote`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **61 tests, 61 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 3
   tools, the request_quote omission absence assertions at
   registry/MCP/CLI levels, plus session handling/reuse, env
   override, the no-Authorization-header assertions under any
   environment, verbatim argument forwarding and result pass-through
   including upstream isError, SSE parsing, error rendering,
   MCP-layer validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/douxcalrisk-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/calriskquote/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/calriskquote/parity-out` → **verdict PASS, parity 100.0**
   (3/3 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
