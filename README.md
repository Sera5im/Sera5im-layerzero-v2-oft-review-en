# LayerZero V2 OFT Review

This repository contains my review notes on the LayerZero V2 OFT path.

<img width="2172" height="724" alt="image" src="https://github.com/user-attachments/assets/7fcc6560-0777-44cc-94a6-564392476c7b" />

## What Is LayerZero OFT?

LayerZero V2 OFT is an omnichain token model.

At a high level, it allows token value to move from one chain to another through a send/receive path:

- value leaves the source-side token path
- a cross-chain message carries the transfer semantics through LayerZero delivery
- value is credited on the destination-side token path

In the plain OFT model reviewed here, the token itself owns that cross-chain send/receive logic.

## What This Review Covers

LayerZero V2 OFT moves token value across chains through an omnichain token model.

In the reviewed OFT path:

- on send, the source-side amount is debited and encoded into an outbound cross-chain message
- after transport delivery, the destination-side amount is credited through the OFT receive path
- if compose mode is enabled, the receive side can continue into an additional compose branch

This review is focused on the OFT contract / application layer path.

## Main Flow

```mermaid
flowchart LR
    A["send(...)"]

    A --> B["_debit(...)"]
    B --> B1["_debitView(...)"]
    B1 --> B11["_removeDust(...)"]
    B --> B2["_burn(...)"]
    B2 --> B3["BURN"]

    A --> C["_buildMsgAndOptions(...)"]
    C --> C1["OFTMsgCodec.encode(...)"]
    C1 --> C11["hasCompose = composeMsg.length > 0"]
    C11 --> C12["msgType = SEND / SEND_AND_CALL"]

    C --> C2["combineOptions(...)"]
    C2 --> C21["enforcedOptions[_eid][_msgType]"]
    C21 --> C22["if enforced.length == 0 -> return extraOptions"]
    C21 --> C23["if extraOptions.length == 0 -> return enforced"]
    C21 --> C24["if extraOptions.length >= 2"]
    C24 --> C25["_assertOptionsType3(...)"]
    C25 --> C26["bytes.concat(enforced, extraOptions[2:])"]
    C24 --> C27["revert InvalidOptions(...)"]

    C --> C3["msgInspector.inspect(message, options)"]

    A --> D["_lzSend(...)"]
    D --> D1["_payNative(...)"]
    D1 --> D2{"lzTokenFee > 0?"}
    D2 -->|yes| D3["_payLzToken(...)"]
    D3 --> D4["endpoint.send(...)"]
    D2 -->|no| D4

    D4 --> T["LayerZero transport / message delivery"]

    T --> R["_lzReceive(...)"]

    R --> R1["_message.sendTo()"]
    R1 --> R11["bytes32ToAddress()"]

    R --> R2["_message.amountSD()"]
    R2 --> R21["_toLD(...)"]
    R21 --> R22["_credit(...)"]
    R22 --> R23["MINT"]

    R --> R3{"_message.isComposed()?"}
    R3 -->|yes| R4["OFTComposeMsgCodec.encode(...)"]
    R4 --> R5["endpoint.sendCompose(...)"]
    R3 -->|no| R6["skip compose path"]

    R --> R7["emit OFTReceived(...)"]

    A --> E["construct OFTReceipt struct"]
    A --> F["emit OFTSent(...)"]
```

## Scope

My current scope here is:

- OFT source-side send logic
- debit / amount-normalization logic
- outbound message construction
- options construction and enforced-options merge logic
- transport-facing send handoff into the LayerZero endpoint
- destination-side receive and credit logic
- optional compose continuation activation
- surrounding config, admin, preview, and helper surface

This review is focused on the contract / application layer rather than a deeper transport-layer review of LayerZero endpoint internals. Because of that, endpoint-level message-delivery internals are treated here only as the transport boundary between the reviewed source-side and destination-side OFT paths.

## OFT vs OFTAdapter

- `OFT` means the token itself is omnichain-aware and owns the cross-chain send/receive logic.
- `OFTAdapter` means an existing token is connected to omnichain transfer logic through a separate adapter contract.

This repository is focused on the `OFT` path.

## Review Structure

- [send-review.md](./send-review.md)
  Main OFT flow review covering the send path, transport boundary, receive path, destination-side credit, and compose continuation branch.

- [out-of-flow-review.md](./out-of-flow-review.md)
  Review of the surrounding config, admin, preview, and helper surface around the main OFT path.
