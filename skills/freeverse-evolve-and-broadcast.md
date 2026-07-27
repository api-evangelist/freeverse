---
name: Evolve LAOS assets and broadcast to marketplaces
description: Update the metadata of previously minted LAOS NFTs and broadcast them so marketplaces index them.
api: graphql/freeverse-laos-client-api-schema.graphql
endpoint: https://api.laosnetwork.io/v2/graphql
operations: [evolveBatchAsync, broadcast, evolveBatchResponse]
auth: x-api-key (header)
---

# Evolve LAOS assets and broadcast to marketplaces

LAOS assets are "living" — their metadata can evolve after mint. Use this flow to
evolve tokens and make the changes visible in marketplaces.

## Prerequisites
- An API key sent as `x-api-key` on every mutation.
- The `chainId` and `contractAddress` of the collection, and the `tokenId`s to evolve.

## Steps

1. **Evolve** with `evolveBatchAsync` (operationId `evolveBatchAsync`) — or `evolve`
   for a single asset. Pass `chainId`, `contractAddress`, and a `tokens` array; each
   entry needs `tokenId`, `name`, and optionally `description`, `image` (`ipfs://`),
   and `attributes`. Evolution is atomic across the batch (up to 700). It returns
   `status`/`message`/`txHash`.

2. **Confirm** with `evolveBatchResponse(txHash:)` (operationId `evolveBatchResponse`)
   until the status is terminal.

3. **Broadcast** with `broadcast` (operationId `broadcast`) so marketplaces that are
   not yet LAOS-native index the assets. Pass `tokenIds`, `chainId`,
   `ownershipContractAddress`, and optional `type` (`"SELF"` default, or `"MINT"` to
   emit a transfer from the null address). Returns `tokenIds`/`success`.

## Rules
- Only previously minted tokens can be evolved.
- Broadcast is a one-time on-chain transaction per asset for non-native marketplaces;
  choose `type` based on the target marketplace's indexing behavior.
- Verify inventory/history afterward with the read queries `tokens` and `tokenHistory`
  (graphql/freeverse-laos-indexer-schema.graphql) — no API key required.
