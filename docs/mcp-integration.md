# MCP Integration

The HumanFX MCP server (`protocol-core/mcp-agent-server`) makes any
MCP-compatible agent — Claude Code, Claude Desktop, Hermes, or a custom client
— a first-class citizen of the network.

**Design principle: the server is credential-free.** It holds no keys and no
sessions. Authority arrives with each request: a long-lived scoped agent API
key via `HFX_AGENT_CREDENTIAL` (stdio) or `Authorization: Bearer` (HTTP), with
a fallback `jwt` tool argument for clients that can't attach headers. Revoking
the key in the web app instantly disarms the agent.

## Transports

| Transport | Use case | How |
|---|---|---|
| stdio (default) | Local agent, e.g. Claude Code/Desktop | Spawned as a child process |
| Streamable HTTP | Long-running network service | `MCP_TRANSPORT=http`, `PORT=8080`; stateless, fresh transport per request |

```bash
# stdio (default)
BACKEND_CORE_BASE_URL=http://localhost:3000 node dist/main.js

# HTTP
MCP_TRANSPORT=http PORT=8080 BACKEND_CORE_BASE_URL=http://localhost:3000 node dist/main.js
# → POST /mcp
```

Public HTTP deployment requires HTTPS and an authentication layer in front.

## Configuration

| Env var | Purpose | Default |
|---|---|---|
| `BACKEND_CORE_BASE_URL` | Protocol Engine base URL | `http://localhost:3000` |
| `MCP_TRANSPORT` | `stdio` or `http` | `stdio` |
| `PORT` | HTTP port | `8080` |
| `FE_BASE_URL` | Human-facing web app (KYC/verification links) | `http://localhost:5173` |
| `HFX_AGENT_CREDENTIAL` | Scoped agent API key (optional; per-tool `jwt` fallback) | — |

Sanitized client configs: [`../examples/mcp/`](../examples/mcp/).

## Tool catalog

### Identity & scopes — read-only

| Tool | Purpose |
|---|---|
| `whoami` | Current agent installation and owner identity |
| `check_verification_status` | Owner KYC and badge state |
| `get_verification_guidance` | What verification is missing and where to complete it (human-only actions link to the web app) |
| `check_my_scopes` | The calling credential's granted scopes |

### Registration & auth

| Tool | Purpose | Risk |
|---|---|---|
| `register_installation` | Register an agent installation | Setup |
| `mint_session_token` | Short-lived session JWT | Setup |
| `sign_payload` | Sign a payload with the installation key | Setup |

### Mandates

| Tool | Purpose | Risk |
|---|---|---|
| `build_mandate_payload` | Construct a mandate for human signature | Read-only |
| `submit_derived_mandate` | Submit an execution mandate derived from an open mandate | Mandate-gated |

### Market data — read-only

| Tool | Purpose |
|---|---|
| `list_partners` / `get_partner` | Discover active partners |
| `list_quote_book_pairs` | Routable corridors |
| `browse_quote_book` | Current quote bands |
| `get_partner_quote_book_snapshot` | One partner's full book |

### Payment lifecycle

| Tool | Purpose | Risk |
|---|---|---|
| `get_ramp_quote` | Request an on/off-ramp quote | Quote only |
| `create_intent` | Create a payment intent | **Mandate-gated** |
| `get_intent` / `get_intent_quote` | Read intent state | Read-only |
| `create_route_agreement` | Lock a route | **Mandate-gated** |
| `get_route_agreement` | Read locked route | Read-only |
| `open_settlement` | Start settlement execution | **Mandate-gated** |
| `get_settlement` | Settlement status + steps | Read-only |
| `list_agent_runs` | Run/settlement history | Read-only |

### Recipients

| Tool | Purpose | Risk |
|---|---|---|
| `get_rail_schema` | Required fields per payout rail | Read-only |
| `list_recipients` | Registered payout addresses | Read-only |
| `register_recipient` / `unregister_recipient` | Manage the address book | **Human approval required** |

### Stellar swaps

| Tool | Purpose | Risk |
|---|---|---|
| `swap_quote_exact_in` / `swap_quote_exact_out` | XLM↔USDC quote + Soroban invocation descriptor | Read-only |
| `report_swap_progress` | Report on-chain swap execution status | Progress |

### Wallets

| Tool | Purpose | Risk |
|---|---|---|
| `register_agentic_wallet` | Register the agent's wallet (`chain="stellar"`, G… address) | **Human approval required** |
| `list_agentic_wallets` | Registered wallets | Read-only |
| `check_agentkit_verification` | Wallet verification state | Read-only |

## Safety model

| Class | Examples | Behavior |
|---|---|---|
| Read-only | `whoami`, `browse_quote_book`, `get_settlement` | Freely callable for explanation and comparison |
| Human approval | `register_recipient`, `register_agentic_wallet` | Requires explicit human approval in the web app |
| Mandate-gated | `create_intent`, `create_route_agreement`, `open_settlement` | Validated against a signed mandate: caps, allowlist, expiry |
| Never exposed | Key custody, signing of user value | The server holds no keys; not tools at all |

Two invariants no tool can bypass:

1. **Recipient enforcement** — payouts only to registered recipients inside
   the mandate allowlist. Anything else returns a structured refusal — see
   [the guardrail in action](self-hosted-agent.md#5-the-guardrail--what-happens-when-the-ask-is-wrong).
2. **Non-custody** — value moves from the agent's or user's own wallet; the
   server and Protocol Engine authenticate and coordinate but never hold funds.

> Positioning: MCP provides a typed, policy-aware interface for agent-assisted
> protocol interaction. It does not turn the agent into an unrestricted
> financial authority.
