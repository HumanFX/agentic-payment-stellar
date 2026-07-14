# Door 1 — The Hosted Agent

The hosted agent is the agent HumanFX runs **for** the user: it lives behind
the web app's chat, holds its keys in the HumanFX agent control plane, and
does everything a payment needs — parsing the ask, comparing quotes, composing
routes, tracking settlement — while every act of financial authority stays
with the human.

Running example throughout: Anh, a furniture exporter in Ho Chi Minh City,
pays her São Paulo supplier Marco R$3,500 — from VND, a corridor no single
provider serves.

> Partner names (VietBridge, BrasilPay, RioRemit) are illustrative demo
> personas backed by mock solvers on Stellar testnet. No real funds move.

## Components involved

| Layer | Component | Role |
|---|---|---|
| UI | `protocol-core/demo-ui` | Chat, confirm cards, intent tracker |
| Agent control plane | `protocol-core/agent-backend` | Key custody, sessions, usage metering, chat relay |
| Agent brain | `agentic-runtime` | LangGraph workflows, human-in-the-loop interrupts, structured actions |
| Protocol | `protocol-core/backend-core` | Quote book, intents, route agreements, mandates, settlements, recipients |
| Rails | Solvers + Soroban contracts | Quote pushes, per-leg execution, Stellar testnet USDC leg |

## 1. A natural-language ask becomes an intent

```text
Send R$3,500 to Marco for the June leather invoice — pay from my VND.
```

No forms, no corridor pickers — the user speaks in money terms, not banking
terms.

**Underneath:** the agent runtime parses the ask and creates an **intent**
object in backend-core: deliver `R$3,500 BRL` to recipient `Marco`, funded
from `VND`. "Marco" is not free text — it resolves against the owner's
**registered recipient book** (`PIX ••1234, registered supplier`) before
anything else happens. An unresolvable recipient stops the flow here.

## 2. Quote discovery and route composition

No provider serves VND→BRL directly, so the agent composes a route across the
network:

```text
Leg 1  VND → USDC   via VietBridge   (best of 3 quotes)
Leg 2  USDC → BRL   via BrasilPay    (best of 4 quotes — 5.42, beat RioRemit 5.40)

All-in: ₫17,100,000 → Marco receives R$3,500
Fees: ₫53,000 (0.31%) · ETA ~12 minutes
Between legs, the money sits as USDC in the USER'S wallet — never HumanFX's.
```

**Underneath:** solvers continuously push quote bands into backend-core's
**quote book**. The agent browses every band on both legs (seven quotes in
this example), ranks them **all-in** (rate + fees), and composes the cheapest
safe two-leg path with `USDC@stellar` as the bridge asset. The comparison
table the user can expand is the raw quote book — the agent's choice is
inspectable, not oracular.

This is the core network effect: liquidity is aggregated across partners, and
the agent — not the user — is the market participant.

## 3. Authorization is explicit, never conversational

The user taps **Confirm & Pay** and a confirm card slides up: amount, all-in
rate, fee, the recipient card, route summary, and a **worst-rate bound** —
completed with hold-to-confirm.

**Underneath:** two invariants.

1. **Chat is never authorization.** The model's text has no execution power.
   Confirmation is a **structured, server-issued action**: backend-core locks
   a **route agreement** (legs, partners, rates, fees, expiry, worst-rate
   bound) and only the signed confirm of *that object* authorizes execution —
   scoped to exactly its contents.
2. **Registered recipients only.** Even a tricked or hallucinating agent
   cannot pay an unregistered address; the worst case is a failed match,
   never a misdirected payout.

## 4. The human's only job: move their own fiat

The UI shows "Your turn: pay ₫17,100,000" with the partner's account, a QR,
a countdown — and the line *"Marco hasn't been charged. Nothing moves until
your payment lands."*

**Underneath:** a **settlement** object opens in `awaiting deposit`. The
fiat-in leg is detected by the partner solver's webhook (simulated in the
demo). This is deliberate design: the only thing a human ever does on HumanFX
is move their own fiat — everything else is the agent's job. HumanFX never
initiates a pull from a user account.

## 5. Execution across legs — non-custodial mid-route

The chat tracks live:

```text
✓ VND received by VietBridge
✓ USDC delivered to your wallet          ← the user's wallet, not HumanFX's
→ Executing leg 2 under your mandate…
✓ USDC sent to BrasilPay
→ BrasilPay paying Marco via PIX — Brazil's instant rail…
```

**Underneath:** each leg is a settlement step driven by solver **step/bill
callbacks** into backend-core. The bridge leg is real Stellar movement: USDC
lands in the user's own testnet wallet, and the agent's authority to forward
it into leg 2 comes from the confirmed route agreement — a narrow,
single-purpose mandate. If anything failed mid-route, the user would hold
USDC in their own wallet with options, not a support ticket.

## 6. Settlement as a first-class, auditable object

The Intent Tracker shows a two-stage timeline: per-leg partner steps,
timestamps, documents and receipt attached.

**Underneath:** the settlement object *is* the audit trail — every callback,
every state transition, timestamped and exportable (`get_settlement`, SSE
stream to the tracker). This is what an accountant sees; this is what an
auditor sees.

## 7. From execution to standing authority

After completion, the agent notices the pattern:

```text
You've paid Marco around the 5th for three months now.
Want me to handle this automatically each month — capped at R$4,000,
only to Marco, and I'll show you the rate before each run?
```

**Underneath:** accepting creates an **open mandate**: monthly cadence,
R$4,000 per-run cap, recipient allowlist of exactly one, rate shown before
each run — revocable with one tap. From then on the agent self-issues a
derived **execution mandate** per run, and backend-core validates every run
against the open mandate's bounds server-side.

This is the point of an agentic network: it doesn't just execute — it takes
the job, inside authority the human explicitly granted and can withdraw.

## Summary — experience vs machinery

| The user sees | The protocol does |
|---|---|
| A sentence in chat | Intent created; recipient resolved against the registered book |
| A composed route with alternatives | Quote-book browse across all solvers; all-in ranking; two-leg composition |
| A hold-to-confirm card | Route agreement locked; structured confirm action (never chat text) |
| "Your turn: pay ₫17,100,000" | Settlement opened; awaiting fiat-in webhook; no pull from user accounts |
| Live leg-by-leg progress | Solver step/bill callbacks; USDC leg on Stellar testnet in the user's wallet |
| A receipt and timeline | Settlement object: timestamped, exportable audit trail |
| "Handle this monthly?" | Open mandate: cadence, cap, allowlist, revocable; per-run derived execution mandates |
