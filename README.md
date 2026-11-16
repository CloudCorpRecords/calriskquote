# calriskquote

Check what CalRisk Insurance Center writes, from your own machine's tool list. calriskquote is a local forwarding client for the hosted CalRisk Insurance Center MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every lookup to the agency service, returning the answers exactly as the service sends them back.

The built surface covers the agency profile (licence, lines and states written, contact details and hours), an appetite check for one line of insurance in one US state, and the link to the agency's own online quoting platform. Nothing is cached, stored or computed locally — the package is a pipe, not an agency.

This is a labelled subset rebuild: the upstream capture enumerates four tools, and three are built here. The remaining one, `request_quote`, sends a person's name, an email or phone, the line, the state and free-text coverage details to the agency's licensed staff for a reply, so it is omitted — not registered, not served in `tools/list`, and not available on the CLI.

## The tools

- **`agency_info`** — Who CalRisk Insurance Center is: licence, the lines and states it writes, how to reach it and its hours. No parameters; call it first.
- **`check_appetite`** — Whether CalRisk Insurance Center writes a given line of insurance in a given US state. Both `line` and `state` are required; `line` is one of `general-liability`, `workers-comp`, `commercial-auto`, `bonds`, `property`, `professional`, and `state` is a two-letter US state such as `CA`.
- **`start_online_quote`** — The link where the client starts a quote with CalRisk Insurance Center on the agency's own quoting platform. An optional `line` may be supplied; the tool returns a link and submits nothing.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/calriskquote mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "calriskquote": {
      "command": "npx",
      "args": ["-y", "@reneturcios/calriskquote", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/calriskquote
docker run --rm ghcr.io/cloudcorprecords/calriskquote check_appetite '{"line":"workers-comp","state":"CA"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
calriskquote agency_info '{}'
calriskquote check_appetite '{"line":"general-liability","state":"CA"}'
calriskquote start_online_quote '{"line":"property"}'
calriskquote --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `DOUXCALRISK_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://douxprocess.com/mcp/a/calrisk-insurance-center`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `calriskquote error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface, asserts the omitted `request_quote` tool is absent at the registry, MCP and CLI levels, exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
