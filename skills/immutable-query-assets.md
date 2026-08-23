---
name: immutable-query-assets
description: Read collections, NFTs, owners, metadata, ERC-20 tokens, activity and pricing from the Immutable Indexer without credentials.
api: immutable-zkevm-api
operations:
  - ListChains
  - ListCollections
  - GetCollection
  - ListNFTs
  - GetNFT
  - ListNFTsByAccountAddress
  - ListNFTOwners
  - ListCollectionsByNFTOwner
  - SearchNFTs
  - SearchStacks
  - ListFilters
  - QuotesForStacks
  - QuotesForNFTs
  - ListActivityHistory
  - ListERC20Tokens
generated: '2026-08-23'
method: generated
source: openapi/immutable-zkevm-openapi.json
---

# Query Immutable Chain assets

These read operations answer **anonymously** — no API key required. Verified 2026-08-23: `GET https://api.immutable.com/v1/chains` returned 200 with no credentials. The anonymous ceiling observed was 5 requests per second (`x-ratelimit-limit: 5, 5;w=1`), so send a key if you need throughput.

## Orientation

1. `ListChains` — `GET /v1/chains`. Start here; every other path is chain-scoped.
2. `ListCollections` / `GetCollection` — `GET /v1/chains/{chain_name}/collections[/{contract_address}]`. Read `contract_type` to learn whether you are dealing with ERC-721 or ERC-1155 before you do anything else.

## Assets

- `ListNFTs` — `GET /v1/chains/{chain_name}/collections/{contract_address}/nfts`
- `GetNFT` — `GET /v1/chains/{chain_name}/collections/{contract_address}/nfts/{token_id}`
- `ListNFTsByAccountAddress` — `GET /v1/chains/{chain_name}/accounts/{account_address}/nfts` — a player's inventory.
- `ListCollectionsByNFTOwner` — `GET /v1/chains/{chain_name}/accounts/{account_address}/collections`
- `ListNFTOwners` — `GET /v1/chains/{chain_name}/collections/{contract_address}/nfts/{token_id}/owners`. `balance` exceeds 1 only for ERC-1155.

There is no server-issued object id to key on. Identity is the `(chain, contract_address, token_id)` triple. Build your cache keys accordingly.

## Search and pricing

- `ListFilters` — `GET /v1/chains/{chain_name}/search/filters/{contract_address}` — discover the metadata attributes available to filter on before you construct a search.
- `SearchNFTs` / `SearchStacks` — `GET /v1/chains/{chain_name}/search/nfts` and `/search/stacks`. A *stack* groups NFTs sharing one metadata identity — that is the unit marketplaces list, not the individual token.
- `QuotesForStacks` / `QuotesForNFTs` — `GET /v1/chains/{chain_name}/quotes/{contract_address}/stacks` and `/nfts` — bulk pricing for a list of ids.

## Replication

Use `ListActivityHistory` — `GET /v1/chains/{chain_name}/activity-history` — for backfills. It is sorted by `updated_at` ascending specifically for time-based replication. Do **not** page `ListActivities` for this; it is not ordered for it.

## Rules

- Pagination is `page_cursor` + `page_size`; follow `page.next_cursor` until it is absent. Never construct a cursor yourself.
- There is no `expand` parameter. Related objects are either embedded whole (a `StackBundle` carries stack + market + listings + bids) or fetched separately.
- `404 RESOURCE_NOT_FOUND` on a collection often means "not indexed yet", not "does not exist".
- All reads are safe to retry. Nothing in this skill mutates state.

## References

- `data-model/immutable-data-model.yml`
- `rate-limits/immutable-rate-limits.yml`
- https://docs.immutable.com/api-reference
