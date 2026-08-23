---
name: immutable-mint-nfts
description: Mint ERC-721 or ERC-1155 game assets on Immutable Chain through the gasless Minting API, then track each request to completion.
api: immutable-zkevm-api
operations:
  - CreateMintRequest
  - GetMintRequest
  - ListMintRequests
  - GetCollection
  - RefreshNFTMetadataByTokenID
generated: '2026-08-23'
method: generated
source: openapi/immutable-zkevm-openapi.json, https://docs.immutable.com/docs/products/asset-contracts/minting-api
---

# Mint NFTs on Immutable Chain

Every operationId below was read out of `openapi/immutable-zkevm-openapi.json`. Nothing here is invented.

## Before you start

- Deploy an ERC-721 or ERC-1155 contract in [Immutable Hub](https://hub.immutable.com) and note its `contract_address`.
- Get a **secret** API key from Hub (Settings → API Keys). The publishable `pk_imapik-` key will be rejected with `401 UNAUTHORISED_REQUEST`.
- Pick the environment. Sandbox is `https://api.sandbox.immutable.com` + `chain_name: imtbl-zkevm-testnet`; production is `https://api.immutable.com` + `chain_name: imtbl-zkevm-mainnet`. **You choose it twice** — once in the host, once in the path — and mismatching them is the most common failure.

## Steps

1. **Confirm the collection exists and is indexed** — `GetCollection`
   `GET /v1/chains/{chain_name}/collections/{contract_address}`
   A `404 RESOURCE_NOT_FOUND` here means the contract has not been indexed yet, not that it does not exist.

2. **Submit the mint** — `CreateMintRequest`
   `POST /v1/chains/{chain_name}/collections/{contract_address}/nfts/mint-requests`
   Header: `x-immutable-api-key: <secret key>`.
   Body carries an `assets[]` array. Each asset needs a `reference_id` you generate and an `owner_address`. `token_id` is required for ERC-1155 (with `amount`) and optional for ERC-721 — omit it to let the system assign one.
   **`reference_id` is the idempotency key.** Re-posting the same value is safe and will not double-mint. Choose something stable and meaningful from your own system, not a timestamp, or you lose the guarantee.

3. **Track it** — `GetMintRequest`
   `GET /v1/chains/{chain_name}/collections/{contract_address}/nfts/mint-requests/{reference_id}`
   `status` moves `pending → succeeded | failed`. On success you get `token_id` and `transaction_hash`; on failure read `error`.
   Prefer the `imtbl_zkevm_mint_request_updated` webhook over polling — see `asyncapi/immutable-webhooks.yml`.

4. **Reconcile a batch** — `ListMintRequests`
   `GET /v1/chains/{chain_name}/collections/{contract_address}/nfts/mint-requests`

5. **Update metadata later** — `RefreshNFTMetadataByTokenID`
   `POST /v1/chains/{chain_name}/collections/{contract_address}/nfts/refresh-metadata`
   Immutable does not watch for on-chain URI updates; you must trigger a refresh explicitly.

## Rules

- **Rate limits count distinct `token_id`s, not tokens.** Standard tier is 200/min with a 2,000 burst; Partner is 2,000/min with 20,000. Read `imx_remaining_mint_requests` and `imx_mint_requests_retry_after` from the response — these are **not** standard `Retry-After` headers.
- **Passing metadata for an ERC-1155 `token_id` that already exists returns `409 CONFLICT_ERROR`.** Check first.
- **A succeeded mint is irreversible.** There is no unmint, no rollback and no reversal window. Idempotency protects you from double-firing; it does not let you take a mint back. Get the recipient and token right before you POST.
- Even when you include metadata in the request, you must **also** host it at `{baseURI}/{token_id}` — some ecosystem partners read from the source.
- Errors carry a required `trace_id` that matches the `x-trace-id` response header. Quote it in any support request.

## References

- `errors/immutable-problem-types.yml`
- `rate-limits/immutable-rate-limits.yml`
- `conventions/immutable-conventions.yml`
- https://docs.immutable.com/docs/products/asset-contracts/minting-api
