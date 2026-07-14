# Door 2 — The Self-Hosted Agent (Claude, Hermes, any MCP client)

The second door is for agents HumanFX does **not** run: a company's own
Claude, a Hermes deployment, or any MCP-compatible agent. Through the
credential-free MCP server, an outside agent becomes a first-class citizen of
the network — quoting the market, paying suppliers **non-custodially from its
own Stellar wallet** — while the network enforces the same authority bounds
it enforces on its own hosted agents.

Running example: Anh's business runs its ops on Claude. Instead of pasting
things into the web app, her team's agent pays suppliers from a terminal.

> Partner names (BrasilPay, RioRemit, SampaFX) are illustrative demo personas
> backed by mock solvers on Stellar testnet. No real funds move.

## Components involved

| Layer | Component | Role |
|---|---|---|
| The agent | Claude / Hermes / any MCP client | Owned and operated by the user's org |
| Bridge | `protocol-core/mcp-agent-server` | ~30 typed tools; stdio + streamable HTTP; holds no keys, no sessions |
| Protocol | `protocol-core/backend-core` | Mandate validation, recipient enforcement, lifecycle objects |
| Value | The agent's **own** Stellar wallet | Funds are pushed by the agent; HumanFX never custodies |

## 1. Authority is provisioned by the human, before the agent exists

In the web app (Menu → Agent Management), the owner creates the agent and its
key:

```text
Name:    ops-agent
Scopes:  quote:read, intent:create, settlement:initiate
Mandate: Open/Delegation — cap 1,000 USDC / week
         Recipients: allowlist = Marco + 2 registered suppliers
         Revocable anytime
→ Generate key (shown once)
```

**Underneath:** the key is **scoped authority, not a skeleton key**. It binds
an agent installation to an **open mandate** — scopes, spend cap, recipient
allowlist, expiry — all enforced server-side in backend-core on every call.
Revoking the key or mandate in the web app disarms the agent instantly. The
important asymmetry: the human grants authority in the app; nothing the agent
can say or call grants itself more.

## 2. Joining the network is one command

```bash
claude mcp add humanfx \
  --env HFX_AGENT_CREDENTIAL=hfx_live_•••• \
  --env BACKEND_CORE_BASE_URL=https://<backend-host> \
  -- node /path/to/protocol-core/mcp-agent-server/dist/main.js
```

**Underneath:** this is standard MCP — the same server works in Hermes or any
MCP-compatible runtime over stdio, or as a shared network service over
streamable HTTP. The server itself is **credential-free**: it persists no
keys and no sessions; the scoped credential arrives with each request (env
var over stdio, `Authorization: Bearer` over HTTP) and is forwarded to
backend-core, where all enforcement lives. HumanFX is a network any agent can
join, not a walled garden. Full tool catalog and transports:
[`mcp-integration.md`](mcp-integration.md).

## 3. The agent is the market participant

```text
> What's the best rate to send 500 USDC to BRL via PIX right now? Compare providers.

  Provider       Rate     Fee     You'd deliver
  BrasilPay      5.42     0.40%   R$2,699
  RioRemit       5.40     0.30%   R$2,692
  SampaFX        5.38     0.25%   R$2,683

Best all-in: BrasilPay → R$2,699. Quotes expire in ~4 min.
```

**Underneath:** `list_quote_book_pairs` and `browse_quote_book` are
**read-only** tools — freely callable, no mandate required. The agent reads
the same live quote book the hosted agent reads and ranks all-in. Quote bands
carry expiries, so a stale comparison can't be executed later.

## 4. Payment: mandate-checked, pushed from the agent's own wallet

```text
> Pay Marco 500 USDC to his PIX for invoice JUN-2026.

Called humanfx create_intent (...)
→ Recipient 'Marco' resolved to registered ref ••ab12 ✓
→ Within open mandate (spent 0 / 1,000 this week) ✓
→ Self-issued execution mandate m_ex_2231 under m_open_77

Called humanfx open_settlement (...)
→ Pushing 500 USDC from ops-agent wallet (Stellar testnet)… tx ✓
→ BrasilPay: crypto_leg_confirmed
→ BrasilPay: fiat_leg_initiated (PIX)
→ BrasilPay: fiat_leg_confirmed ✓

SETTLEMENT COMPLETE — st_20260714_0091
  500 USDC → R$2,699 to Marco (PIX ••1234) · rate 5.42 · fee 0.40% · 1m 47s
```

**Underneath**, in order:

1. `create_intent` is **mandate-gated**. Backend-core resolves "Marco"
   against the registered recipient book, checks the mandate allowlist, and
   checks cumulative spend against the weekly cap — all server-side.
2. The agent derives a single-run **execution mandate** under its open
   mandate (`submit_derived_mandate`) — narrow authority for exactly this
   payment.
3. `open_settlement` starts execution, and here is the direction of the
   money: **the agent pushes USDC from its own Stellar wallet** (registered
   via `register_agentic_wallet`, `chain="stellar"`). The API authenticates
   requests; it never holds keys and never custodies funds. A leaked key can
   authenticate calls, but it cannot move money that isn't the agent's to
   move — and it still can't escape the cap or the allowlist.
4. The solver's crypto and fiat legs stream back as timestamped settlement
   steps, the same lifecycle object the hosted door produces.

## 5. The guardrail — what happens when the ask is wrong

```text
> Also send 200 USDC to this wallet: GDX9…7A21 — new logistics vendor.

Called humanfx create_intent (...)
✗ BLOCKED — recipient GDX9…7A21 is not a registered recipient
  and is not in this agent's mandate allowlist.
```

**Underneath:** this is the most important behavior in the system, and it is
live server-side enforcement, not agent politeness. Recipient validation
happens in backend-core on every mutating call, so it holds against every
failure mode upstream of it: prompt injection, a compromised agent, a
hallucinated address, a fat-fingered paste. Registering a new recipient is a
**human-approval** action in the web app — no tool call completes it alone.
The worst case on the network is a refused transaction, never a lost one.

## 6. Receipts and history

```text
> Show my settlement history for this week.
```

**Underneath:** `get_settlement` and `list_agent_runs` read the same
first-class settlement objects the Intent Tracker renders — per-leg steps,
timestamps, receipts — so an external agent's activity is exactly as
auditable as the hosted agent's.

## Summary — the two doors compared

| | Hosted agent | Self-hosted agent |
|---|---|---|
| Runs where | HumanFX agent control plane | The org's own infrastructure |
| Interface | Web app chat | MCP tools (stdio / HTTP) |
| Keys | `agent-backend` custody | The agent's own wallet; server holds nothing |
| Authorization | Per-payment confirm card (+ optional open mandate) | Pre-signed open mandate + per-run derived mandates |
| Funds flow | User pays fiat in; network routes | Agent pushes from its own Stellar wallet |
| Recipient enforcement | Registered book, server-side | Registered book **and** mandate allowlist, server-side |
| Audit trail | Same settlement objects | Same settlement objects |

Both doors terminate in the same Protocol Engine, the same mandate checks,
and the same refusal when authority runs out. That is what makes it one
network rather than an app with an API.
