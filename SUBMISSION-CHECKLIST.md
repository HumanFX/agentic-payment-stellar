# Submission Checklist (internal — DELETE before making this repo public)

## Security scrub in protocol-core (`stella-hackathon`) — do FIRST

- [ ] `demo-ui/.env.local` is **tracked in git** — untrack it, add to
      `.gitignore`, and rotate the Google client ID / World app ID if they
      matter.
- [ ] `backend-core/.env.test` is **tracked in git** — untrack or confirm it
      contains only local test values.
- [ ] Confirm no `stellar-contracts/deployments/*.json` with issuer/distributor
      **secrets** are committed anywhere in branch history.
- [ ] Grep branch history for `STELLAR_TREASURY_SECRET`, `DEPLOYER_PRIVATE_KEY`,
      `hfx_live_`, real JWTs.
- [ ] `backend-core/supported-rails-bank/` (third-party PII QR screenshots) is
      gitignored — confirm nothing slipped into history.
- [ ] Decide whether legacy EVM pieces (`escrow-contracts`,
      `recurrence-contracts`, `hfx-wallet-cli`) stay visible — they're fine but
      off-story; consider a README note that they're the EVM lineage.

## Repo access

- [ ] Make `protocol-core` and `agentic-runtime` public, or grant the
      organizer read access — the pinned-commit links in README.md assume it.
- [ ] Verify both pinned commit URLs resolve once access exists:
      protocol-core `513fc7cf…`, agentic-runtime `03cef2ee…`.

## Fill the TBDs

- [ ] Hosted demo UI URL → README "Demo links" table.
- [ ] Demo video (Loom/YouTube) → README + `assets/demo-thumbnail.png`.
- [ ] Screenshots per `assets/screenshots/README.md`.
- [ ] Confirm the `claude mcp add` invocation in `docs/self-hosted-agent.md`
      matches the final deployed backend host.

## Final pass

- [ ] Re-read `docs/status.md` against what the demo actually shows — no
      claim drift.
- [ ] Run every link in README.md.
- [ ] Delete this file.
