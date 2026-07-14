# API Examples (sanitized)

Illustrative calls against a local Protocol Engine (`backend-core`,
`http://localhost:3000`). Shapes are representative of the payment lifecycle;
consult the pinned `protocol-core` commit for authoritative DTOs.

> All examples use testnet assets and demo values. Never commit real
> credentials; `$HFX_JWT` is a short-lived session token.

## Browse the quote book

```bash
curl -s "http://localhost:3000/quote-book?pair=USDC@stellar/PHP&rail=instapay" \
  -H "Authorization: Bearer $HFX_JWT"
```

## Create a payment intent

```bash
curl -s -X POST http://localhost:3000/intents \
  -H "Authorization: Bearer $HFX_JWT" -H "Content-Type: application/json" \
  -d @intent-request.json
```

See [`intent-request.json`](intent-request.json).

## Read a settlement

```bash
curl -s http://localhost:3000/settlements/st_20260714_0091 \
  -H "Authorization: Bearer $HFX_JWT"
```

Representative response: [`settlement-status.json`](settlement-status.json).

## Quote an on-chain swap (XLM → USDC)

```bash
curl -s "http://localhost:3000/swap/quote?side=exact_in&sell=XLM@stellar&buy=USDC@stellar&amount=100.0000000" \
  -H "Authorization: Bearer $HFX_JWT"
```

Returns the quoted amount plus the Soroban `fixed_rate_swap` invocation
descriptor for the caller's wallet to sign and submit (7-decimal amounts).
