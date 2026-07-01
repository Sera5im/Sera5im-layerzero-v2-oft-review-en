# Out-of-Flow Review

This review covers the surrounding config, admin, preview, and helper surface around the main OFT send/receive path.

It is not about the normal token-movement path itself, but about the functions that configure, support, or preview that path.

## 1. OAppCore.setPeer(...)

```solidity
function setPeer(uint32 _eid, bytes32 _peer) public virtual onlyOwner {
    _setPeer(_eid, _peer);
}
```

What it does:

- updates the stored peer for a selected destination eid
- defines which remote counterpart is trusted for that route

Invariants:

- this function must not be callable by arbitrary users

## 2. OAppCore._getPeerOrRevert(...)

```solidity
function _getPeerOrRevert(uint32 _eid) internal view virtual returns (bytes32) {
    bytes32 peer = peers[_eid];
    if (peer == bytes32(0)) revert NoPeer(_eid);
    return peer;
}
```

What it does:

- loads the configured peer for the selected destination eid
- reverts if no peer is configured

Invariants:

- send path must not continue without a configured peer for the selected destination eid

## 3. OAppCore.setDelegate(...)

```solidity
function setDelegate(address _delegate) public onlyOwner {
    endpoint.setDelegate(_delegate);
}
```

What it does:

- updates the endpoint-side delegate configuration

Invariants:

- this function must not be callable by arbitrary users

## 4. OFTCore.setMsgInspector(...)

```solidity
function setMsgInspector(address _msgInspector) public virtual onlyOwner {
    msgInspector = _msgInspector;
    emit MsgInspectorSet(_msgInspector);
}
```

What it does:

- updates the optional message-inspector hook used by the OFT path

Invariants:

- this function must not be callable by arbitrary users

## 5. OFTCore.quoteOFT(...)

```solidity
function quoteOFT(
    SendParam calldata _sendParam
) external view virtual returns (OFTLimit memory oftLimit, OFTFeeDetail[] memory oftFeeDetails, OFTReceipt memory oftReceipt) {
    uint256 minAmountLD = 0;
    uint256 maxAmountLD = type(uint64).max;

    oftLimit = OFTLimit(minAmountLD, maxAmountLD);
    oftFeeDetails = new OFTFeeDetail[](0);

    (uint256 amountSentLD, uint256 amountReceivedLD) = _debitView(
        _sendParam.amountLD,
        _sendParam.minAmountLD,
        _sendParam.dstEid
    );

    oftReceipt = OFTReceipt(amountSentLD, amountReceivedLD);
}
```

What it does:

- previews the token-side result of the OFT path
- returns token limits and the debit preview receipt

Invariants:

- quoteOFT(...) must not distort the amount preview returned by _debitView(...)

## 6. OFTCore.quoteSend(...)

```solidity
function quoteSend(
    SendParam calldata _sendParam,
    bool _payInLzToken
) external view virtual returns (MessagingFee memory msgFee) {
    (, uint256 amountReceivedLD) = _debitView(
        _sendParam.amountLD,
        _sendParam.minAmountLD,
        _sendParam.dstEid
    );

    (bytes memory message, bytes memory options) = _buildMsgAndOptions(_sendParam, amountReceivedLD);

    return _quote(_sendParam.dstEid, message, options, _payInLzToken);
}
```

What it does:

- previews the transport-side fee for the future OFT send path
- prices the payload that the real send path would prepare

Invariants:

- quoteSend(...) must quote the fee for the same message/options payload that the real send path would build

## 7. OAppCore.isPeer(...)

```solidity
function isPeer(uint32 _eid, bytes32 _peer) public view virtual override returns (bool) {
    return peers[_eid] == _peer;
}
```

What it does:

- checks whether the provided peer matches the stored peer for the selected eid

## 8. OAppPreCrimeSimulator._lzReceiveSimulate(...)

```solidity
function _lzReceiveSimulate(
    Origin calldata _origin,
    bytes32 _guid,
    bytes calldata _message,
    address _executor,
    bytes calldata _extraData
) internal virtual override {
    _lzReceive(_origin, _guid, _message, _executor, _extraData);
}
```

What it does:

- forwards simulation input into the normal internal receive logic
- exposes a simulate-oriented helper surface around the receive path
