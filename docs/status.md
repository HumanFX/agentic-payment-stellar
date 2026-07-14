# Status — Real vs Mocked vs Planned

One page of complete honesty about what this submission is.

## Real and running (Stellar testnet)

| Component | Detail |
|---|---|
| Soroban contracts | `fixed_rate_swap` (reserve-backed XLM↔USDC) and `mock_usdc` (7-decimal SEP-41) built with Soroban SDK v27, deployed to testnet with reproducible scripts |
| On-chain swaps | Demo UI Swap flow + MCP swap tools return real Soroban invocation descriptors; swaps execute on testnet |
| Stellar wallet auth | SIWE-style challenge/verify for G… addresses; agentic wallet registry with `chain="stellar"` |
| Stellar settlement leg | Alix solver pushes USDC over Horizon testnet as the crypto leg, with escrow sweep + reconciliation |
| Protocol lifecycle | Quote book, intents, route agreements, mandates, settlements — live backend objects with per-step audit trail |
| Mandate enforcement | Server-side scope, cap, allowlist, and expiry checks on every mutating call |
| Recipient enforcement | Payouts blocked to any address outside the registered book + allowlist (the [blocked-recipient refusal](self-hosted-agent.md#5-the-guardrail--what-happens-when-the-ask-is-wrong) is live behavior, not a mock) |
| MCP server | ~30 tools, stdio + streamable HTTP, credential-free design |
| Demo UI | On-ramp, off-ramp, swap, limit orders, recipients, live tracker (SSE) |
| Agent control plane | Key custody, sessions, usage metering, chat relay (`agent-backend`) |
| Agent runtime | LangGraph workflows, human-in-the-loop interrupts, durable events (`agentic-runtime`) |

## Mocked or simulated

| Component | What's actually happening |
|---|---|
| USDC asset | Mock issuer on testnet — not Circle-issued USDC |
| Swap rate | Fixed by contract admin (demo default 1 XLM = 0.19 USDC) — not market-derived |
| Partner quotes/fills | `mock-solver` publishes the asked rate, so execution is instant and deterministic |
| Partner identities | VietBridge, BrasilPay, RioRemit, SampaFX are demo personas backed by mock solvers |
| Fiat legs | Simulated via solver step/bill webhooks; the PHP payout edge calls the Alix Pay **sandbox** |
| Hosted-agent replies | The scripted demo conversation runs against the live backend, but wording is curated for the demo |

## Planned — not in this submission

| Item | Where it's tracked |
|---|---|
| Stellar Anchor SEP-6 / SEP-24 deposit & withdrawal | [`roadmap.md`](roadmap.md), Phase 2 |
| Real USDC + market-rate routing (DEX/AMM or partner books) | Phase 3 |
| Mainnet settlement | Phase 3–4 |
| Production hardening (reconciliation, monitoring, audits) | Phase 4 |

## Explicitly not claimed

- No real fiat settlement occurs anywhere in the demo.
- No real user funds are used.
- No production readiness is claimed.
- Mainnet-looking labels, addresses, or receipts in screenshots and scripts do
  not imply live settlement.
