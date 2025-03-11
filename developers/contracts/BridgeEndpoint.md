# BridgeEndpoint
- Location: `xlink/packages/contracts/bridge-solidity/contracts`
- Deployed contracts: [BridgeEndpoint](0x84254dA34abE4678017A5Bf78506B48490ce4547), [BridgeEndpointWithSwap]().

This technical document provides a detailed overview of the Bridge Endpoint in the Ethereum blockchain. The Bridge Endpoint facilitates communication between two blockchain networks by acting as the entry and exit point for assets moving along the Cross Chain Bridge. It passes messages between chains in the form of events, trigers contract calls, processes token transfers and validates and executes the unwrapping of tokens. The Bridge Endpoint functionalitiy is implemented across two contracts, `BridgeEndpoint` and `BridgeEndpointWithSwap`. `BridgeEndpointWithSwap` extends `BridgeEndpoint` and implements the necessary functionality to source liquidity from external aggregators.

This functionality is implemented and distributed across the following contracts:

- `BridgeEndpoint`: 
- `BridgeEndpointWithSwap`: extends `BridgeEndpoint` and integrates swaps during a bridge transfer.

## Storage

### `registry`

| Data     | Type   |
| -------- | ------ |
| Variable | `BridgeRegistry` |

Holds a reference to the BridgeRegistry contract, which manages approved tokens, relayers, and validators.

### `pegInAddress`

| Data     | Type   |
| -------- | ------ |
| Variable | `address` |

The address where tokens are deposited before bridging.

### `timeLock`

| Data     | Type   |
| -------- | ------ |
| Variable | `ITimeLock` |

Manages locked transactions that require a delay before execution. Tokens exceeding the timelock threshold are not immediately sent to the user. Instead, the timeLock contract holds them until the delay expires. After the delay, the timeLock contract releases the tokens.

### `timeLockThreshold`

| Data     | Type   |
| -------- | ------ |
| Variable | `uint256` |

The minimum token amount that triggers a timelock. By default, the value is set to 0.

### `timeLockThresholdByToken`
| Data     | Type   |
| -------- | ------ |
| Variable | `mapping(address => uint256)` |

Custom timelock thresholds for different tokens.

### `mapping(bytes32 => OrderPackage) public unwrapSent`
| Data     | Type   |
| -------- | ------ |
| Variable | `mapping(bytes32 => OrderPackage)` |

Stores information about unwrap transactions.

## Data Types

#### `OrderPackage`

Holds details of a pending unwrap operation.

```lisp
struct OrderPackage {
  address recipient;
  address token;
  uint256 amount;
  bool sent;
}
```

#### `SignaturePackage`

Contains signatures from validators to prove a transaction is legitimate. It is used for verifying cross-chain transactions.

```lisp
struct SignaturePackage {
  bytes32 orderHash;
  address signer;
  bytes signature;
}
```

## Modifiers

onlyApprovedToken(token): Ensures the token is approved in the registry.

onlyApprovedRelayer(): Ensures the caller is an approved relayer.

notWatchlist(recipient): Prevents transactions to watchlisted addresses.

nonReentrant: Protects against reentrancy attacks.

onlyAllowlisted: Ensures only allowed addresses can execute certain functions.

## Features

#### `sendMessageWithToken`
This function transfers a token across chains, deducts a fee, and emits an event. This is the function that is called when the user executes a transaction. The user deposits tokens on the bridge contract in the source chain. The contract checks that the token is approved (since sendMessageWithToken has the onlyApprovedToken modifier). This function calls `_transfer`, which performs validations. Finally, it emits a `SendMessageWithTokenEvent` containing the sender's address, the token and amount sent, the deducted fee and the encoded paylaod (destination details). Afterwards, off-chain validators sign the transaction.

##### Parameters
```lisp
(address token, uint256 amount, bytes calldata payload)
```

#### `sendMessage`

Emits an event with a message and ETH value.

##### Parameters
```lisp
(address token, uint256 amount, bytes calldata payload)
```

#### `transferToUnwrap`

This function is called by a relayer once that `sendMessageWithToken` is called, with the purpose of sending tokens to a recipient. Relayers will listen for the corresponding `SendMessageWithTokenEvent` on the source chain. A relayer calls `transferToUnwrap` providing the token, recipient and amount, a salt (unique transaction identifier, usually the source chain transaction hash), and an array of validator signatures (proofs). The contract validates the transaction and generates a EIP-712 hash.

##### Parameters
```lisp
address token,
address recipient,
uint256 amount,
bytes32 salt,
SignaturePackage[] calldata proofs
```

#### `finalizeUnwrap`

Completes an unwrap transaction and releases tokens to the recipient. If the token transfer was delayed (timelocked), the unwrap is finalized manually. The function loops through each orderHash and calls _finalizeUnwrap(orderHash), which ensures the transaction is not already completed, transfers tokens from unwrapSent mapping to the recipient and marks the order as sent. Finally, it emits `FinalizeUnwrapEvent.`

##### Parameters
```lisp
(bytes32[] calldata orderHash)
```

#### `transferToSwap`
###### _(in contract BridgeEndpointWithSwap)_`

This function executes a swap before bridging tokens. If the token is burnable, it mints it before swapping. It validates tokens and relayer permissions and generates a unique order hash for the swap. If the token is burnable, it then mints the token and calls `_executeSwap()` and emits `TransferToSwapEvent`. If it is not burnable, it saves swap details in the `swapSent` mapping and emits `SwapOrderCreated` so the swap can be finalized later.

##### Parameters
```lisp
address target,
address tokenIn,
address tokenOut,
uint256 amountIn,
uint256 amountOutMin,
bytes calldata swapPayload,
bytes calldata bridgePayloadSuccess,
bytes calldata bridgePayloadFailure,
bytes32 salt,
SignaturePackage[] calldata proofs
```

#### `finalizeSwap`
###### _(in contract BridgeEndpointWithSwap)_`
This function is used when a token is not burnable, leading transferToSwap to store the swap order instead of executing it immediately. It ensures input arrays are valid, and it loops through each orderHash and calls `_finalizeSwap()`.

##### Parameters
```lisp
bytes32[] calldata orderHashes,
bytes[] calldata swapPayloads
```

### Governance Features

#### `setTimeLock`

Updates the contract managing timelocks.

##### Parameters
```lisp
address _timeLock
```
#### `setTimeLockThreshold`

Sets the global timelock threshold

##### Parameters
```lisp
uint256 _timeLockThreshold
```

#### `setTimeLockThresholdByToken`

Sets a custom timelock threshold per token.

##### Parameters
```lisp
(address token, uint256 _timeLockThreshold)
```

### Relevant Internal Functions

#### `_transfer`

Transfers a token from the sender, deducts fees, and sends the remainder to the bridge.

##### Parameters
```lisp
ddress token, uint256 amount
```

#### `_validateOrder`

Verifies if an order is legitimate by checking validator signatures.

##### Parameters
```lisp
(bytes32 orderHash, SignaturePackage[] calldata proofs)
```

#### `_finalizeUnwrap`

Completes an unwrap transaction by transferring tokens to the recipient.

##### Parameters
```lisp
(bytes32 orderHash)
```

#### `_finalizeSwap`
###### _(in contract BridgeEndpointWithSwap)_`

This function executes a stored swap order. It is called by `finalizeSwap` to retrieve a stored swap  order and to execute the swap. It checks if the order exists and has not been executed, transfers `amountIn` tokens from the sender to the `BridgeEndpointWithSwap` contract and calls `_executeSwap` to perform the swap. Finally, it marks the order as sent and emits a `SwapOrderFinalized` event.

##### Parameters
```lisp
bytes32 orderHash, bytes memory swapPayload
```

#### `_executeSwap`
###### _(in contract BridgeEndpointWithSwap)_`

This function uses `swapExecutor` to execute the swap logic. It approves `swapExecutor` to spend `tokenIn`, calls `swapExecutor.executeSwap()` to attempt the swap. If the swap succeeds, it either burns the swapped tokens or prepares them for transfer. To transfer the tokens, `tokenOut` is sent to `pegInAddress` for bridging and a `SendMessageWithTokenEvent` is emitted. If the swap fails, the error is logged via `SwapExecutorError`, approvals are revoked and a `SendMessageWithTokenEvent` with `bridgePayloadFailure` is emitted. In either case, the function will burn `tokenIn` tokens if applicable. 

##### Parameters
```lisp
address tokenIn,
address tokenOut,
address target,
uint256 amountIn,
uint256 amountOutMin,
bytes memory swapPayload,
bytes memory bridgePayloadSuccess,
bytes memory bridgePayloadFailure
```