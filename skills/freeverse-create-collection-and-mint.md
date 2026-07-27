---
name: Create a LAOS collection and mint NFTs
description: Create a bridgeless-minting collection on LAOS and mint NFTs to owners on Ethereum or Polygon.
api: graphql/freeverse-laos-client-api-schema.graphql
endpoint: https://api.laosnetwork.io/v2/graphql
operations: [createCollection, mintAsync, mintResponse]
auth: x-api-key (header)
---

# Create a LAOS collection and mint NFTs

Use the LAOS Network GraphQL API to stand up a collection and mint NFTs whose
ownership is enforced on an established chain (Ethereum or Polygon) via bridgeless
minting.

## Prerequisites
- An API key from the LAOS Foundation (email info@laosnetwork.io). Send it on every
  write mutation as the `x-api-key` request header. Read queries need no key.
- A supported `chainId`: `"1"` (Ethereum) or `"137"` (Polygon).
- The key's associated web3 address must hold gas (LAOS/POL/ETH); testnet keys are pre-funded.

## Steps

1. **Create the collection** with `createCollection` (operationId `createCollection`).
   Pass `name`, `symbol`, and `chainId`. The response returns `contractAddress`
   (the ERC721 on the target chain), `laosAddress` (the sibling LAOS collection),
   and `batchMinterAddress`. Persist `contractAddress`.

2. **Mint NFTs** with `mintAsync` (operationId `mintAsync`) — recommended over the
   synchronous `mint` for large batches. Pass `chainId`, the `contractAddress` from
   step 1, and a `tokens` array (up to 700). Each token needs `mintTo` (owner
   addresses), `name`, and optionally `description`, `image` (an `ipfs://` URI), and
   `attributes` (`trait_type`/`value`). The whole batch is atomic — if it fails, no
   tokens are minted. `mintAsync` returns `txHash`/`status`/`message`.

3. **Confirm** by polling `mintResponse(txHash:)` (operationId `mintResponse`) until
   `status` is terminal; read the returned `tokenIds` and `receipt`.

## Rules
- Batches are atomic and capped at 700 tokens; split larger jobs.
- There is no idempotency-key; use the returned `txHash`/`trackingId` to dedup and
  track submitted work (see conventions/freeverse-conventions.yml).
- Errors come back in the GraphQL `errors` array; blockchain outcomes surface on the
  payload `status`/`message` fields (see errors/freeverse-error-codes.yml).
