# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted CalRisk Insurance Center MCP server. The three built captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `agency_info`, `check_appetite`, `start_online_quote`. This is a labelled subset of the captured 4-tool surface: `request_quote` is omitted because it sends a person's name, contact details and coverage details to the agency's licensed staff for a reply, and its absence is asserted in the tests at the registry, MCP and CLI levels.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `calriskquote error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `DOUXCALRISK_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
