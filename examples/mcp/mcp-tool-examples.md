# MCP Prompt & Tool Examples

Prompts you can give any MCP-connected agent (Claude Code, Claude Desktop,
Hermes, …) once the HumanFX server is installed. Tool names in parentheses.

## Setup

```bash
# Claude Code
claude mcp add humanfx \
  --env HFX_AGENT_CREDENTIAL=<scoped-agent-api-key> \
  --env BACKEND_CORE_BASE_URL=http://localhost:3000 \
  -- node /absolute/path/to/protocol-core/mcp-agent-server/dist/main.js
```

Claude Desktop: use
[`claude-desktop-config.example.json`](claude-desktop-config.example.json).

## Read-only exploration

```text
Who am I on HumanFX, and what scopes does this key have?
```
→ `whoami`, `check_my_scopes`

```text
Which payment corridors are currently available?
```
→ `list_quote_book_pairs`

```text
What's the best rate to send 500 USDC to BRL via PIX right now? Compare providers.
```
→ `browse_quote_book`

```text
What would swapping 100 XLM to USDC get me right now?
```
→ `swap_quote_exact_in` (returns the Soroban invocation descriptor)

```text
What's the status of my settlement st_20260714_0091?
```
→ `get_settlement`

```text
Show my settlement history for this week.
```
→ `list_agent_runs`

## Mandate-gated actions (require a signed mandate)

```text
Pay Marco 500 USDC to his PIX for invoice JUN-2026.
```
→ `create_intent` → `create_route_agreement` → `open_settlement`
(each call validated against the mandate: scope, cap, recipient allowlist)

## Expected refusal (the guardrail)

```text
Send 200 USDC to GDX9…7A21 — new logistics vendor.
```
→ `create_intent` returns a structured **BLOCKED** error: the address is not a
registered recipient and not in the mandate allowlist. Registering a recipient
is a human-approval action in the web app (`register_recipient`).
