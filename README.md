# LayerZero OFT Review

This repository contains my review notes on the LayerZero OFT path.

## What This Review Covers

LayerZero OFT moves token value across chains through an omnichain token model.

At a high level, the reviewed path is:

- source-side debit
- outbound message construction
- transport-facing handoff into LayerZero delivery
- destination-side receive and credit logic

This review is focused on the contract / application layer OFT path rather than a deep review of endpoint-level message-delivery internals.

## Scope

My current scope here is:

- source-side OFT send logic
- debit / amount-normalization logic
- outbound message and options construction
- transport-facing send handoff
- destination-side receive / credit logic
- surrounding config, admin, preview, and helper surface

LayerZero endpoint-level delivery internals are treated here only as the transport boundary between the reviewed contract-layer paths.

## OFT vs OFTAdapter

- `OFT` means the token itself is omnichain-aware and owns the cross-chain send/receive logic.
- `OFTAdapter` means an existing token is connected to omnichain transfer logic through a separate adapter contract.

This repository is focused on the `OFT` path.

## Review Structure

- [send-review.md](./send-review.md)
  Main OFT flow review covering send-side and receive-side path reasoning.

- [out-of-flow-review.md](./out-of-flow-review.md)
  Review of config, admin, preview, and helper surface around the main OFT path.

## Method Note

This review is organized around:

- flow decomposition
- function-level reasoning
- invariant verification
- surrounding admin/config surface checks
