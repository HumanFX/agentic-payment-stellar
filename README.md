# HumanFX — Agentic Cross-Border Payments on Stellar

> A human-supervised agentic FX network: AI agents discover quotes, compose
> multi-leg payment routes, and settle over Stellar — inside signed, revocable
> mandates, with every payout locked to registered recipients.

![Demo](https://img.shields.io/badge/demo-Stellar%20testnet-blue)
![Soroban](https://img.shields.io/badge/Soroban-judges--only%20verification-orange)
![MCP](https://img.shields.io/badge/MCP-integrated-green)
![Fiat](https://img.shields.io/badge/fiat%20legs-simulated%2Fsandbox-yellow)
![Production](https://img.shields.io/badge/production-not%20ready-red)

**Stellar APAC Hackathon submission — showcase repository.**
This public repo is the entry point for the story, architecture, and demo
guides. The implementation and deployment identifiers are private. Hackathon
judges can verify the implementation through an unlisted demo, a live technical
walkthrough, and a judges-only evidence pack.

> **Evaluation note:** private implementation is not presented as public proof.
> Replace every `<PLACEHOLDER>` below before submission and give organizers a
> working verification route.

---

## The story: Anh pays Marco

Anh runs a furniture-export business in Ho Chi Minh City. Every month she pays
Marco, her leather supplier in São Paulo, R$3,500. Today that means banks, two
intermediaries, a bad rate, and three days.

On HumanFX she does it in two ways — **two doors, one network**:

1. **Hosted agent (web app).** Anh types *"Send R$3,500 to Marco —
   pay from my VND."* No single provider serves VND→BRL, so her agent composes
   the route: VND → USDC (best of 3 quotes) → BRL (best of 4 quotes), with the
   USDC resting **in her own Stellar wallet** between legs. She confirms one
   signed authorization card and pays her VND. The agent does everything else.
   → [`docs/hosted-agent.md`](docs/hosted-agent.md)

2. **Self-hosted agent (Claude, Hermes, any MCP client).** Anh's ops team runs
   Claude in a terminal. One `claude mcp add` command connects it to the
   network with a scoped key: three allowlisted suppliers, 1,000 USDC/week
   cap, revocable anytime. Claude quotes the market, pays Marco
   non-custodially from its own wallet — and when asked to pay an
   **unregistered** address, the network refuses that payout request.
   → [`docs/self-hosted-agent.md`](docs/self-hosted-agent.md)

The point of an agentic network: it doesn't just execute — it takes the job,
inside authority the human explicitly grants and can revoke.

---

## Demo links

| Resource | Link |
|---|---|
| Hosted demo UI | `<JUDGES_ONLY_DEMO_URL>` |
| Unlisted demo video | `<UNLISTED_DEMO_VIDEO_URL>` |
| Judges-only evidence pack | `<PRIVATE_EVIDENCE_PACK_URL>` |
| Live technical review | `<JUDGE_REVIEW_BOOKING_OR_CONTACT>` |
| Hosted agent explained | [`docs/hosted-agent.md`](docs/hosted-agent.md) |
| Self-hosted agent explained (MCP) | [`docs/self-hosted-agent.md`](docs/self-hosted-agent.md) |
| Architecture | [`docs/architecture.md`](docs/architecture.md) |
| Stellar integration detail | [`docs/stellar-integration.md`](docs/stellar-integration.md) |
| MCP integration guide | [`docs/mcp-integration.md`](docs/mcp-integration.md) |
| Status matrix | [`docs/status.md`](docs/status.md) |

## Private implementation references

These repositories intentionally return no public source and cannot be shared.
The pinned commits identify the reviewed internal versions; implementation
claims must be supported through the code-free evidence pack and live demo.

| Area | Access | Branch | Pinned commit |
|---|---|---|---|
| Protocol Engine, demo UI, MCP server, Soroban contracts, solvers | Private; not shared | `stella-hackathon` | `513fc7cf7866b8afc7b57169d8a63130bdcdbf64` |
| Agent runtime (LangGraph workflows, human-in-the-loop) | Private; not shared | `development/1.0.0` | `03cef2ee7c81b5b524ab010180e7b4dfc8acc08c` |

Key paths inside `protocol-core`:

```text
backend-core/            Protocol Engine — quote book, intents, route agreements,
                         mandates, settlements, identity, Stellar wallet linking
demo-ui/                 React demo app — on-ramp, off-ramp, on-chain swap,
                         limit orders, live intent tracker
mcp-agent-server/        Credential-free MCP server (stdio + streamable HTTP)
stellar-contracts/       Soroban contracts: fixed_rate_swap, mock_usdc + deploy scripts
agent-backend/           Agent control plane — key custody, sessions, chat relay
mock-solver/             Mock partner solver (instant fills, demo rates)
proactive-alix-solver/   Solver on the Stellar USDC rail with sandboxed PHP payout
```

---

## What runs on Stellar today

The team represents that the following is implemented on **Stellar testnet**.
Because source and deployment identifiers are private, judges should validate
these claims through the private evidence pack and live technical walkthrough:

- **Soroban contracts** (`stellar-contracts/`, Soroban SDK v27):
  `fixed_rate_swap` — a reserve-backed XLM↔USDC swap contract; and
  `mock_usdc` — a 7-decimal SEP-41 test asset (deployed alongside a classic
  asset + SAC). Deploy and operate via the included scripts.
- **On-chain swaps**: the demo UI's Swap flow and the MCP `swap_quote_exact_in`
  / `swap_quote_exact_out` tools return real Soroban contract invocation
  descriptors for `XLM@stellar ↔ USDC@stellar`.
- **Stellar wallet linking**: backend-core authenticates Stellar wallets with
  a SIWE-style challenge and registers agentic wallets with `chain="stellar"`
  (G… addresses).
- **Stellar settlement rail**: the Alix solver pushes USDC over Stellar
  (Horizon testnet) as the crypto leg of crypto→fiat settlements.

Full detail: [`docs/stellar-integration.md`](docs/stellar-integration.md).

## Development status

| Feature | Team status | Public evidence | Judge verification |
|---|---|---|---|
| Web demo UI | Implemented | Documentation only | `<JUDGES_ONLY_DEMO_URL>` |
| Hosted agent chat | Implemented | Documentation only | `<UNLISTED_DEMO_VIDEO_URL>` |
| Soroban contracts | Deployed to testnet | Contract ID withheld | `<PRIVATE_CONTRACT_PROOF>` |
| On-chain XLM↔USDC swap | Implemented | Documentation only | `<PRIVATE_TRANSACTION_PROOF>` |
| Stellar wallet linking / auth | Implemented | Documentation only | `<PRIVATE_AUTH_DEMO>` |
| MCP server + Stellar tools | Implemented | Tool catalog | `<PRIVATE_MCP_DEMO>` |
| Quote → intent → route → settlement lifecycle | Implemented | Example payloads | `<PRIVATE_LIFECYCLE_DEMO>` |
| Mandates + recipient allowlist enforcement | Implemented | Expected behavior | `<PRIVATE_GUARDRAIL_DEMO>` |
| USDC asset | ⚠️ Mock | Testnet issuer | Not circle-issued USDC |
| Partner quotes/fills | ⚠️ Mocked | Mock solver | Publishes asked rate; instant fills |
| Fiat settlement | ⚠️ Simulated | Sandbox | PHP payout edge via Alix sandbox |
| Stellar Anchor (SEP-6/24) | 🚧 Planned | Not connected | Next milestone — see roadmap |
| Mainnet settlement | ❌ Not enabled | — | Testnet only |
| Production readiness | ❌ Not claimed | — | Hackathon demonstration |

## Important demo disclaimer

> This showcase runs on **Stellar testnet** against mocked or sandboxed
> partners. The USDC asset is a test-issued mock; partner quotes and fills come
> from a mock solver; fiat legs are simulated (the PHP payout edge runs against
> a payment-provider sandbox). Partner names in the demo script (VietBridge,
> BrasilPay, RioRemit, SampaFX) are illustrative personas backed by mock
> solvers. **No real user funds move.** Stellar Anchor (SEP-6/SEP-24)
> integration is planned and not yet connected. This demo must not be used for
> production payments.

## Why it's safe by construction

- **Chat is never authorization.** Financial mutations require structured,
  server-issued actions confirmed by a human or covered by a signed mandate.
- **Mandates are scoped authority**: per-run caps, weekly caps, recipient
  allowlists, expiry — revocable with one tap.
- **Registered recipients only.** The documented policy rejects payout requests
  to recipients outside the registered book and mandate allowlist.
- **Separated custody roles.** User-value keys are not held by the Protocol
  Engine or MCP server. The hosted agent control plane does hold hosted-agent
  operational keys; exact signing boundaries are documented for judges in the
  private architecture review.

Full model: [`docs/security-and-trust.md`](docs/security-and-trust.md).

## Documentation map

| Doc | Contents |
|---|---|
| [`docs/architecture.md`](docs/architecture.md) | Components, data flow, diagram |
| [`docs/stellar-integration.md`](docs/stellar-integration.md) | Contracts, testnet setup, asset conventions |
| [`docs/hosted-agent.md`](docs/hosted-agent.md) | The hosted agent, step by step, with what's happening underneath |
| [`docs/self-hosted-agent.md`](docs/self-hosted-agent.md) | Self-hosted agents (Claude/Hermes) via MCP, incl. the guardrail refusal |
| [`docs/mcp-integration.md`](docs/mcp-integration.md) | Tool catalog, transports, safety classes |
| [`docs/security-and-trust.md`](docs/security-and-trust.md) | Mandates, allowlists, non-custody |
| [`docs/status.md`](docs/status.md) | Real vs mocked vs planned, in detail |
| [`docs/roadmap.md`](docs/roadmap.md) | Anchor integration and mainnet path |
| [`examples/`](examples/) | Sanitized MCP configs and API payloads |

## License

[MIT](LICENSE) — applies to this showcase repository's documentation and
examples. Implementation repositories carry their own licenses.
