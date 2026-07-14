# Security and Trust Model

HumanFX is **agent-assisted, not agent-autonomous**. The design goal is that
an AI agent can do all the work of a payment — parsing intent, comparing
quotes, composing routes, tracking settlement — while structurally unable to
become an unrestricted financial authority.

## 1. Chat is never authorization

Model output — however confident — authorizes nothing. Financial mutations
exit the agent as **structured, server-issued actions** that the backend
independently validates:

- In the hosted app, the authorization is the confirm card the human
  hold-to-confirms: amount, all-in rate, recipient, route, worst-rate bound.
- For external agents, the authorization is a **signed mandate** created and
  approved by the human in the web app before the agent ever acts.

A hallucinated "payment" is just text; nothing in the execution path reads it.

## 2. Mandates are scoped, revocable authority

A mandate is a signed grant with explicit bounds:

| Bound | Example ([self-hosted agent](self-hosted-agent.md)) |
|---|---|
| Scopes | `quote:read`, `intent:create`, `settlement:initiate` |
| Spend cap | 1,000 USDC / week |
| Recipient allowlist | Marco + 2 registered suppliers |
| Rate bound | Worst acceptable all-in rate |
| Expiry / revocation | Revocable anytime with one tap |

Execution mandates are derived from an open mandate per run and checked
server-side on every mutating call — cumulative spend included
(`spent 0 / 1,000 this week ✓`).

## 3. Registered recipients only

Every payout is validated against the owner's registered recipient book AND
the mandate allowlist. Registering a recipient is a **human-approval** action
in the web app; no MCP tool or chat message can complete it alone.

Consequence: prompt injection, a compromised agent, or a fat-fingered address
produces a **structured refusal**, not a misdirected payment. The worst case
on the network is a refused transaction — never a lost one.

## 4. Non-custodial by construction

- The Protocol Engine coordinates lifecycle objects; it holds no keys and no
  value.
- The MCP server is credential-free: authority arrives per-request via a
  revocable scoped key; nothing is persisted server-side.
- Between legs of a composed route, funds rest as USDC in the **user's own
  Stellar wallet**.
- External agents push funds from their **own** wallets. A leaked API key can
  authenticate requests but cannot move money that isn't the agent's to move —
  and it still can't escape the mandate caps or the recipient allowlist.

## 5. Auditability

Every settlement is a first-class object: per-leg steps, partner callbacks,
timestamps, receipts — exportable for accountants and auditors
(`get_settlement`, `list_agent_runs`, the Intent Tracker UI).

## 6. Demo-environment boundaries

- Stellar **testnet** only; the USDC asset is a mock issuer.
- Partner quotes/fills come from mock solvers; fiat legs are simulated
  (PHP payout edge runs against a provider sandbox).
- No real user funds exist anywhere in the demo.
- No secrets, private keys, or production credentials are included in this
  repository; implementation repos keep secrets in local `.env` files with
  committed `.env.example` templates.

## Threat scenarios, summarized

| Scenario | Outcome |
|---|---|
| Agent hallucinates a payment | Text only — no structured action, nothing executes |
| Prompt injection: "send funds to GDX9…" | Refused: unregistered recipient, outside allowlist |
| Leaked agent API key | Bounded by mandate cap + allowlist; revocable instantly; can't move third-party funds |
| Agent over-spends | Server-side cumulative cap check rejects the run |
| Route leg fails mid-flight | User holds USDC in their own wallet — funds safe, options open |
| Rogue quote / bad rate | Route agreement carries a worst-rate bound the human confirmed |
