---
name: Immutable
description: Use when building games, marketplaces, or web3 applications that need authentication, NFT contracts, trading, player data, or blockchain integration on Immutable Chain. Reach for this skill when implementing Passport wallets, deploying smart contracts, creating NFT listings, minting assets, or setting up player analytics.
metadata:
    mintlify-proj: immutable
    version: "1.0"
---

# Immutable Skill

## Product Summary

Immutable is a growth platform for games and web3 apps. It provides authentication (Passport), NFT contracts (ERC-721, ERC-1155, ERC-20), a decentralized orderbook for trading, player analytics (Audience CDP), and Immutable Chain—a gaming-optimized zkEVM blockchain with fast finality and gas sponsorship.

**Key files and commands:**
- SDKs: `@imtbl/auth`, `@imtbl/wallet`, `@imtbl/orderbook`, `@imtbl/blockchain-data`, `@imtbl/minting-backend`
- Configuration: Immutable Hub (https://hub.immutable.com) for API keys, Passport clients, contracts
- Networks: Testnet (Chain ID 13473, RPC https://rpc.testnet.immutable.com) and Mainnet (Chain ID 13371, RPC https://rpc.immutable.com)
- Primary docs: https://docs.immutable.com

## When to Use

Reach for this skill when:
- **Authentication & Wallets**: Implementing Passport login (Google, Apple, email) or embedded wallet operations
- **NFT Contracts**: Deploying ERC-721 (unique NFTs), ERC-1155 (multi-token), or ERC-20 (currencies)
- **Minting**: Server-side NFT minting via Minting API (200–2000+ NFTs/min depending on tier)
- **Trading**: Creating listings, bids, collection bids, trait bids, or metadata bids via Orderbook
- **Player Data**: Ingesting player profiles, tracking events, or running engagement campaigns via Audience CDP
- **Blockchain Integration**: Deploying to Immutable Chain, bridging from Ethereum, or querying on-chain data
- **Checkout**: Adding onramp (fiat-to-crypto), swap, bridge, or primary sales flows

## Quick Reference

### API Keys

| Key Type | Use | Security |
|----------|-----|----------|
| **Publishable Key** | Client-side SDK init, browser apps | Safe to expose |
| **Secret Key** | Server-side minting, admin ops, webhooks | Keep private, use env vars |

Manage in Hub: **Settings** → **API Keys**. Rotate immediately if exposed.

### SDK Packages (Install Only What You Need)

| Package | Purpose |
|---------|---------|
| `@imtbl/auth` | Authentication & sessions |
| `@imtbl/wallet` | Embedded wallets & transactions |
| `@imtbl/orderbook` | NFT trading (listings, bids) |
| `@imtbl/blockchain-data` | On-chain data queries (Indexer) |
| `@imtbl/minting-backend` | Server-side minting |
| `@imtbl/contracts` | Smart contract ABIs & types |
| `@imtbl/config` | Environment configuration |

### Network Configuration

| Property | Testnet | Mainnet |
|----------|---------|---------|
| **Chain ID** | 13473 | 13371 |
| **RPC** | https://rpc.testnet.immutable.com | https://rpc.immutable.com |
| **Currency** | tIMX | IMX |
| **Explorer** | explorer.testnet.immutable.com | explorer.immutable.com |

### Orderbook Order Types

| Type | Creator | Example |
|------|---------|---------|
| **Listing** | Seller | "Selling Sword #123 for 1 IMX" |
| **Bid** | Buyer | "Offering 0.5 IMX for Sword #123" |
| **Collection Bid** | Buyer | "Offering 0.3 IMX for any Sword" |
| **Trait Bid** | Buyer | "Offering 0.4 IMX for Swords with Rarity=Legendary" |
| **Metadata Bid** | Buyer | "Offering 0.4 IMX for Swords named Cool Dragon" |

### Minting API Rate Limits

| Tier | NFTs/min | Burst | Use Case |
|------|----------|-------|----------|
| Standard | 200 | 2,000 | Testing, small mints |
| Partner | 2,000 | 20,000 | Production games |
| Enterprise | Custom | Custom | High-volume launches |

## Decision Guidance

### When to Use Passport vs External Wallets

| Scenario | Use Passport | Use MetaMask/WalletConnect |
|----------|--------------|---------------------------|
| Game with new players | ✓ Zero friction (Google/Apple login) | ✗ Requires wallet knowledge |
| Existing crypto users | ✓ Still works, unified wallet | ✓ Familiar experience |
| Mobile game | ✓ Native support, no extensions | ✗ Limited mobile support |
| Cross-game assets | ✓ One wallet across all games | ✗ Per-app wallets |
| Pre-approved transactions | ✓ Native clients only | ✗ Not supported |

### When to Use ERC-721 vs ERC-1155

| Scenario | ERC-721 | ERC-1155 |
|----------|---------|----------|
| Unique items (characters, weapons) | ✓ | ✗ |
| Stackable items (potions, ammo) | ✗ | ✓ |
| Multiple copies of same item | ✗ | ✓ |
| Partial order fills needed | ✗ | ✓ |
| Simpler contract | ✓ | ✗ |

### When to Use Minting API vs Direct Transactions

| Scenario | Minting API | Direct Txn |
|----------|-------------|-----------|
| Server-side minting | ✓ | ✗ |
| Batch minting (100+) | ✓ | ✗ |
| Gas sponsorship needed | ✓ | ✗ |
| Nonce/batching complexity | ✓ (handled) | ✗ (manual) |
| Client-side minting | ✗ | ✓ |

### When to Use Checkout Flows

| User State | Best Flow |
|-----------|-----------|
| No crypto, needs to buy | **Onramp** (Transak: credit card → IMX) |
| Has Ethereum L1 tokens | **Bridge** (transfer to zkEVM) |
| Has wrong token on zkEVM | **Swap** (QuickSwap: exchange tokens) |
| Needs specific token | **Fund** (smart routing: onramp + bridge + swap) |

## Workflow

### 1. Set Up Project Credentials

1. Go to https://hub.immutable.com and create a project
2. Create a **Publishable Key** (for client-side SDK init)
3. Create a **Secret Key** (for server-side minting, keep private)
4. Create a **Passport Client** (for authentication) with redirect URI matching your app
5. Store keys in `.env` files (never commit secrets)

### 2. Implement Passport Authentication

1. Install: `npm install @imtbl/auth @imtbl/wallet`
2. Initialize Auth with your Client ID and redirect URI
3. Call `connectWallet()` on user action (button click)
4. Users see Passport login (Google, Apple, email)
5. On callback, extract wallet address and access token
6. Store session for subsequent API calls

### 3. Deploy NFT Contract

1. Go to Hub → **Contracts** → **Deploy**
2. Choose ERC-721 (unique) or ERC-1155 (multi-token)
3. Set name, symbol, royalties (0.5%–10%), metadata URI
4. Deploy (no code required)
5. Copy contract address for minting and trading

### 4. Mint NFTs (Server-Side)

1. Install: `npm install @imtbl/minting-backend`
2. Use Secret Key in request header: `x-immutable-api-key`
3. POST to `https://api.immutable.com/v1/mint` with:
   - `contract_address`
   - `token_id` (unique per NFT)
   - `to` (recipient wallet)
   - `amount` (1 for ERC-721, any for ERC-1155)
4. Poll `/mint-requests/{reference_id}` to check status
5. Minting is gasless (gas sponsorship)

### 5. Create Orderbook Listings

1. Install: `npm install @imtbl/orderbook`
2. Get user's wallet address from Passport
3. Call `prepareListing()` with:
   - `makerAddress` (seller)
   - `sell` (NFT: contract, tokenId, amount)
   - `buy` (price: NATIVE IMX or ERC-20)
4. Sign the order (gasless, no gas cost)
5. Submit with `createListing()`
6. Order is live on orderbook (visible across all marketplaces)

### 6. Fill Orders (Buy NFTs)

1. Query listings via `listAllListings()` or search
2. Call `fulfillOrder()` with listing ID and buyer address
3. SDK returns required actions (approvals + fill transaction)
4. Execute each action via wallet
5. Trade settles on-chain immediately

### 7. Ingest Player Data (Audience CDP)

1. Identify players after Passport login (use email or user ID)
2. Send events via Web SDK, Unity SDK, or REST API
3. Events feed into Audience CDP profiles
4. Use Analytics to segment players
5. Run campaigns via Engage (email, push, SMS)

## Common Gotchas

- **Redirect URI mismatch**: Must match exactly (http vs https, trailing slash, port). Verify in Hub.
- **Secret key exposed**: Rotate immediately in Hub. Never commit to Git or expose in client code.
- **Approval pattern**: Sellers must approve Seaport contract once per collection before listing. Buyers must approve ERC-20 spend before filling orders with tokens.
- **Order status async**: Orders transition PENDING → ACTIVE asynchronously. Build UI to handle delays.
- **Partial fills**: Only ERC-1155 supports partial fills. ERC-721 orders are all-or-nothing.
- **Maker vs Taker fees**: Maker fees set at order creation (cannot change). Taker fees set at fulfillment (flexible).
- **Nonce management**: Use Minting API to avoid manual nonce tracking. Direct transactions require careful nonce handling.
- **Gas sponsorship**: Only works with Passport wallets and pre-approved contracts. External wallets pay gas.
- **Token allowlisting**: Swap and bridge widgets only show allowlisted tokens. Contact support for new tokens.
- **Metadata hosting**: Self-hosted metadata requires high availability. Server downtime breaks NFT display.
- **Deprecated contracts**: Signed Zone v1 sunset May 2025. Use v2 (SDK auto-updates).
- **Rate limits**: Minting API has per-tier limits. Standard tier: 200 NFTs/min. Monitor response headers.

## Verification Checklist

Before submitting work:

- [ ] API keys stored in `.env`, never in code
- [ ] Publishable key used client-side, Secret key server-side only
- [ ] Redirect URI matches Hub configuration exactly
- [ ] Passport Client ID and environment (Sandbox vs Production) correct
- [ ] Contract deployed and address verified in Hub
- [ ] Minting API requests include Secret Key header
- [ ] Orderbook listings signed (gasless) before submission
- [ ] Order approvals handled (seller approval for listings, buyer approval for ERC-20 fills)
- [ ] Callback page handles OAuth redirect and extracts access token
- [ ] Error handling for AUTHENTICATION_ERROR, WALLET_CONNECTION_ERROR, INVALID_CONFIGURATION
- [ ] Network configuration matches environment (Testnet RPC for Sandbox, Mainnet RPC for Production)
- [ ] Metadata URIs are accessible and valid JSON
- [ ] Royalties set to at least 0.5% for Trading Rewards eligibility
- [ ] Player identification (email, user ID) sent before events for Audience CDP

## Resources

- **Comprehensive page navigation**: https://docs.immutable.com/llms.txt
- **API Keys & Security**: https://docs.immutable.com/docs/guides/advanced-setup/api-keys
- **Passport Authentication**: https://docs.immutable.com/docs/products/passport/authentication
- **Orderbook Trading**: https://docs.immutable.com/docs/products/orderbook/overview
- **Asset Contracts & Minting**: https://docs.immutable.com/docs/products/asset-contracts/overview
- **Immutable Chain Network**: https://docs.immutable.com/docs/products/immutable-chain/overview
- **TypeScript SDK**: https://docs.immutable.com/docs/sdks/typescript/overview

---

> For additional documentation and navigation, see: https://docs.immutable.com/llms.txt