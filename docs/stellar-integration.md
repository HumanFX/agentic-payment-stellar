# Stellar Integration

Everything below is implemented and running on **Stellar testnet** in this
submission (pinned commit
[`513fc7c`](https://github.com/HumanFX/protocol-core/tree/513fc7cf7866b8afc7b57169d8a63130bdcdbf64),
branch `stella-hackathon`).

## Network context

| Setting | Value |
|---|---|
| Network | Stellar **testnet** |
| Soroban RPC | `https://soroban-testnet.stellar.org` |
| Horizon | `https://horizon-testnet.stellar.org` |
| Passphrase | `Test SDF Network ; September 2015` |
| Assets | `XLM@stellar`, `USDC@stellar` (mock issuer), `PHP` (fiat) |
| Decimals | 7 (Stellar-native convention, used end-to-end) |

## Soroban contracts — `protocol-core/stellar-contracts`

Rust, Soroban SDK v27, Cargo workspace, with Node/`@stellar/stellar-sdk`
deploy scripts.

### `fixed_rate_swap`

A reserve-backed fixed-rate XLM↔USDC swap:

- Stores admin, the native XLM SAC address, the USDC SAC address, and a rate
  in USDC-per-XLM (demo default `1 XLM = 0.19 USDC`).
- Admin funds reserves on both sides; users swap against the reserves at the
  posted rate.
- Quoting and swapping are exposed through backend-core's swap-quote service
  and the MCP `swap_quote_exact_in` / `swap_quote_exact_out` tools, which
  return the contract invocation descriptor the agent's wallet signs and
  submits.

A fixed rate keeps the demo deterministic. The contract boundary is the same
one a production AMM/DEX-router integration would use.

### `mock_usdc`

A 7-decimal SEP-41-style test asset (admin mint, balances, allowances,
transfer, burn). In practice the deploy scripts favor a **classic Stellar
asset + its SAC** for mock USDC — closer to how real USDC exists on Stellar —
with the custom contract as an alternative.

### Scripts

```text
scripts/build.sh                     Build contracts (wasm)
scripts/deploy-xlm-sac.sh            Deploy/lookup the native XLM SAC
scripts/deploy-mock-usdc.sh          Deploy mock USDC
scripts/setup-mock-usdc-sac.mjs      Classic asset + SAC route (recommended)
scripts/create-mock-usdc-trustline.* Trustlines for holders
scripts/mint-mock-usdc.sh            Mint test USDC
scripts/deploy-fixed-rate-swap.sh    Deploy the swap contract
scripts/set-swap-rate.sh             Set/adjust the rate
scripts/add-swap-reserve.sh          Fund reserves
scripts/quote-fixed-rate-swap.sh     Quote a swap
scripts/swap-fixed-rate.sh           Execute a swap
```

All scripts target testnet. Deployment artifacts (including generated issuer
and distributor keys) are written locally and must never be committed.

## Stellar in the Protocol Engine — `backend-core`

- **Wallet linking / auth**: `modules/identity/auth/stellar-siwe.verifier.ts`
  and `linked-wallet.service.ts` implement a SIWE-style challenge/verify flow
  for Stellar wallets; agentic wallets register with `chain="stellar"`
  (G… addresses).
- **Swap quotes**: `modules/intent/swap-quote.service.ts` quotes XLM↔USDC
  against the fixed-rate contract, respecting the 7-decimal convention.
- **Currency catalog** carries the Stellar asset references
  (`XLM@stellar`, `USDC@stellar`) and testnet issuer metadata.

## Stellar in the solver — `proactive-alix-solver`

The solver's crypto leg runs over Stellar:

- Pushes USDC on Horizon **testnet** from a treasury account
  (`BASE_CCY=USDC@stellar`).
- Escrow sweeper and reconciler track the on-chain leg; lifecycle step/bill
  callbacks report progress into the settlement object.
- The fiat leg (PHP payout) executes against the Alix Pay **sandbox**.

## Stellar in the MCP server

`swap_quote_exact_in`, `swap_quote_exact_out`, and `report_swap_progress`
return/consume Soroban invocation descriptors so any MCP-capable agent can
execute on-chain swaps with its **own** wallet — the server never holds keys.
`register_agentic_wallet` accepts `chain="stellar"`.

## What is intentionally not real (yet)

- `USDC@stellar` is a **mock issuer** on testnet, not Circle-issued USDC.
- The swap rate is fixed by an admin, not market-derived.
- Fiat rails are simulated or sandboxed — see [`status.md`](status.md).
- Anchor SEP-6/SEP-24 deposit/withdrawal is the next milestone — see
  [`roadmap.md`](roadmap.md).
