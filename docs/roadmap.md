# Roadmap — From Testnet Demo to Stellar-Native Settlement

## Phase 1 — This submission (done)

- Agentic payment flow end-to-end on Stellar **testnet**: intent → quotes →
  composed route → mandate-checked execution → settlement tracking.
- Soroban contracts (`fixed_rate_swap`, `mock_usdc`) with reproducible deploy
  scripts.
- Stellar wallet linking and agentic wallet registry.
- MCP server making any MCP-capable agent a network participant.
- Mandate + registered-recipient enforcement as live server-side behavior.
- Mock partner solvers; simulated/sandboxed fiat legs.

## Phase 2 — Stellar Anchor integration

- Integrate Anchor **SEP-6 / SEP-24** deposit and withdrawal flows as solver
  rails, replacing simulated fiat legs corridor by corridor.
- Anchor transaction status polling/webhooks mapped into the existing
  settlement step model (the step/bill callback contract already fits this).
- Idempotent Anchor event ingestion.
- **SEP-12** KYC handoff — HumanFX's passported-KYC model mapped onto Anchor
  customer requirements.
- Stellar **memo** handling end-to-end (already represented in the branch).

## Phase 3 — Real assets, real rates

- Replace mock USDC with Circle-issued USDC on Stellar; asset trustline
  management in onboarding.
- Market-derived pricing: path payments / DEX / AMM routing alongside partner
  quote books, replacing the fixed-rate demo contract at the same boundary.
- Additional corridors with real Anchor partners across APAC (VND, PHP, THB,
  IDR are the design targets).
- Testnet → mainnet promotion behind feature flags.

## Phase 4 — Production hardening

- Confirmation and reconciliation rules per rail; failure/retry handling.
- Operational monitoring, alerting, and audit logs.
- Security and privacy review (mandate signing, key rotation, webhook
  authentication).
- Limited pilot with sandboxed Anchors before any real-money launch.

---

> Positioning: Anchor integration is a planned extension, not a completed
> feature of this submission. What this submission proves is the harder part —
> that an agentic payment network can be safe by construction (mandates,
> allowlists, non-custody) while settling over Stellar. Anchors plug into the
> solver boundary that already exists.
