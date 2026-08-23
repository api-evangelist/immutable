---
name: immutable-trade-nfts
description: Create, discover, fulfil and cancel NFT orders on the Immutable Orderbook, including the soft-versus-hard cancellation decision.
api: immutable-zkevm-api
operations:
  - CreateListing
  - ListListings
  - GetListing
  - CreateBid
  - CreateCollectionBid
  - CreateTraitBid
  - CreateMetadataBid
  - fulfillment_data
  - CancelOrders
  - ListTrades
  - GetTrade
generated: '2026-08-23'
method: generated
source: openapi/immutable-zkevm-openapi.json, https://docs.immutable.com/docs/products/orderbook/cancel-orders
---

# Trade NFTs on the Immutable Orderbook

The Orderbook is a Seaport deployment. Orders are EIP-712 signed off-chain by the maker's wallet, then submitted; creating an order costs no gas.

## Create

- **Listing (sell)** — `CreateListing`, `POST /v1/chains/{chain_name}/orders/listings`
- **Bid on one NFT** — `CreateBid`, `POST /v1/chains/{chain_name}/orders/bids`
- **Bid on any NFT in a collection** — `CreateCollectionBid`, `POST /v1/chains/{chain_name}/orders/collection-bids`
- **Bid on NFTs matching traits** — `CreateTraitBid`, `POST /v1/chains/{chain_name}/orders/trait-bids`
- **Bid on NFTs matching a metadata id** — `CreateMetadataBid`, `POST /v1/chains/{chain_name}/orders/metadata-bids`

A seller must approve the Seaport contract once per collection before the first listing. A buyer paying in ERC-20 must approve the spend before filling.

## Discover

`ListListings`, `ListBids`, `ListCollectionBids`, `ListTraitBids`, `ListMetadataBids`, and the single-order getters `GetListing` / `GetBid` / `GetCollectionBid` / `GetTraitBid` / `GetMetadataBid`.

Pagination is cursor-based everywhere: `page_cursor` + `page_size` in, `page.next_cursor` out. Sort with `sort_by` + `sort_direction`.

## Fulfil

`fulfillment_data`, `POST /v1/chains/{chain_name}/orders/fulfillment-data` — returns signed fulfilment data for a list of order ids and their fees. Execute the returned actions (approvals, then the fill) through the wallet. Settlement is immediate on-chain.

Confirm with `ListTrades` / `GetTrade`.

## Cancel — read this before you choose

`CancelOrders`, `POST /v1/chains/{chain_name}/orders/cancel`

| | Soft cancel | Hard cancel |
|---|---|---|
| Cost | Free, gasless | Costs gas |
| Effect | Orderbook stops issuing fulfilment data | Order blacklisted in the settlement contract |
| Guarantee | **90-second race window** | Definitive |

The 90 seconds run from when the orderbook *issued* fulfilment data, not from when you cancelled. Inside that window a buyer holding those details can still complete the trade at the agreed price.

Use a hard cancel when the item is high-value (>$1000), the counterparty looks suspicious, bots are trading against you, or the order already shows `pending_cancellations`. Otherwise soft cancel is fine.

**Batch limit: 20 orders per soft-cancel transaction.** Split anything larger.

## Rules

- Only ERC-1155 orders support partial fills; ERC-721 orders are all-or-nothing.
- Maker fees are fixed at order creation and cannot be changed afterwards. Taker fees are set at fulfilment.
- Orders transition `PENDING → ACTIVE` asynchronously. Do not assume an order is live the moment the POST returns.
- A completed trade cannot be reversed. There is no refund or void operation anywhere in this contract.

## References

- `conventions/immutable-conventions.yml` (see `reversibility`)
- `errors/immutable-problem-types.yml`
- https://docs.immutable.com/docs/products/orderbook/overview
