---
name: Read Ronin chain data via JSON-RPC
description: Query balances, blocks, transactions, and contract state on the Ronin blockchain using its Ethereum-compatible JSON-RPC API.
api: Ronin JSON-RPC API
base_url: https://api.roninchain.com/rpc
operations: [eth_chainId, eth_blockNumber, eth_getBalance, eth_getTransactionByHash, eth_getTransactionReceipt, eth_call]
---

# Read Ronin chain data via JSON-RPC

Ronin is an EVM chain, so it speaks the standard Ethereum JSON-RPC protocol. The
public mainnet endpoint `https://api.roninchain.com/rpc` accepts unauthenticated
read/broadcast calls; for higher limits or archive data use the authenticated
gateway at `https://api-gateway.skymavis.com` with an `X-API-KEY`.

All calls are HTTP POST with `Content-Type: application/json` and a JSON-RPC 2.0
envelope: `{"jsonrpc":"2.0","method":"<method>","params":[...],"id":1}`.

## Steps

1. **Confirm the network.** Call `eth_chainId`. Ronin mainnet returns `0x7e4`
   (chain 2020). Abort if it does not match the network you intend to read.
2. **Get the chain head.** Call `eth_blockNumber` to learn the latest block
   (hex). Use `latest` as the block tag for subsequent reads.
3. **Read an account balance.** Call `eth_getBalance` with
   `params: ["<0x-address>", "latest"]`; divide the returned hex wei by 1e18 for RON.
4. **Look up a transaction.** Call `eth_getTransactionByHash` with the tx hash,
   then `eth_getTransactionReceipt` for its status (`0x1` success / `0x0` failed)
   and logs.
5. **Read contract state.** Call `eth_call` with
   `params: [{"to":"<contract>","data":"<abi-encoded call>"}, "latest"]` for
   view functions (e.g. ERC-721 `ownerOf`, ERC-20 `balanceOf`).

## Rules
- JSON-RPC errors come back as `{"error":{"code","message"}}` — surface `message`.
  `-32000` typically means execution reverted or insufficient funds.
- Rate limiting on the public endpoint returns HTTP 429; back off, or switch to a
  keyed gateway endpoint for production throughput.
- For NFT/account/token queries that need indexing rather than raw node reads,
  prefer the Skynet Web3 API (`X-API-KEY`) over building it on JSON-RPC.
