# BridgeEndpoint
- Location: `xlink/packages/contracts/bridge-solidity/contracts`
- Deployed contracts: [BridgeEndpoint](0x84254dA34abE4678017A5Bf78506B48490ce4547), [BridgeEndpointWithSwap](0x84254dA34abE4678017A5Bf78506B48490ce4547).

This technical document provides a detailed overview of the Bridge Endpoint in the Ethereum blockchain. The Bridge Endpoint facilitates communication between two blockchain networks by acting as the entry and exit point for assets moving along the Cross Chain Bridge. It passes messages between chains in the form of events, triggers contract calls, processes token transfers and validates and executes the unwrapping of tokens. The Bridge Endpoint functionality is implemented across two contracts, `BridgeEndpoint` and `BridgeEndpointWithSwap`. `BridgeEndpointWithSwap` extends `BridgeEndpoint` and implements the necessary functionality to source liquidity from external aggregators.

This functionality is implemented and distributed across the following contracts:

- `BridgeEndpoint`: the base contract that facilitates bridging operations.
- `BridgeEndpointWithSwap`: extends `BridgeEndpoint` and integrates swaps during a bridge transfer.

## Storage

### `registry`

| Data     | Type   |
| -------- | ------ |
| Variable | `BridgeRegistry` |

Stores a reference to the `BridgeRegistry` contract, which manages approved tokens, relayers, and validators.

### `pegInAddress`

| Data     | Type   |
| -------- | ------ |
| Variable | `address` |

The address where tokens are deposited before bridging.

### `timeLock`

| Data     | Type   |
| -------- | ------ |
| Variable | `ITimeLock` |

Manages locked transactions that require a delay before execution. Tokens exceeding the timelock threshold are not immediately sent to the user. Instead, the timeLock contract holds them until the waiting period expires and then releases the tokens.

### `timeLockThreshold`

| Data     | Type   |
| -------- | ------ |
| Variable | `uint256` |

The minimum token amount that triggers a timelock. By default, the value is set to `0`.

### `timeLockThresholdByToken`
| Data     | Type   |
| -------- | ------ |
| Variable | `mapping(address => uint256)` |

Custom timelock thresholds for different tokens.

### `unwrapSent`
| Data     | Type   |
| -------- | ------ |
| Variable | `mapping(bytes32 => OrderPackage)` |

Stores unwrap transactions that need to be finalized. It stores a mapping of `OrderPackage` structs, which contain a flag indicating whether the unwrap has been completed.

### `swapExecutor`
###### _(only present in BridgeEndpointWithSwap)_

| Data     | Type   |
| -------- | ------ |
| Variable | `SwapExecutor` |

Holds a reference to SwapExecutor, which is the contract that executes swaps.

### `swapSent`
###### _(only present in BridgeEndpointWithSwap)_

| Data     | Type   |
| -------- | ------ |
| Variable | `mapping(bytes32 => SwapOrderPackage)` |

A mapping to track swap orders. `bytes32` is a unique hash of the swap parameters, as calculated by `transferToSwap`.

## Data Types

#### `OrderPackage`

Holds details of a pending unwrap operation.

```solidity
struct OrderPackage {
  address recipient;
  address token;
  uint256 amount;
  bool sent;
}
```

#### `SignaturePackage`

Contains signatures from validators to prove a transaction is legitimate. It is used for verifying cross-chain transactions.

```solidity
struct SignaturePackage {
  bytes32 orderHash;
  address signer;
  bytes signature;
}
```

#### `SwapOrderPackage`
###### _(only present in BridgeEndpointWithSwap)_

A struct that stores details of a swap transaction.

```solidity
struct SwapOrderPackage {
  address target;
  address tokenIn;
  address tokenOut;
  uint256 amountIn;
  uint256 amountOutMin;
  bytes bridgePayloadSuccess;
  bytes bridgePayloadFailure;
  bool sent;
}
```

## Modifiers

- `onlyApprovedToken(token)`: ensures the token is approved in the registry.
- `onlyApprovedRelayer()`: ensures the caller is an approved relayer.
- `notWatchlist(recipient)`: prevents transactions to watchlisted addresses.
- `nonReentrant`: protects against reentrancy attacks.
- `onlyAllowlisted`: ensures only allowed addresses can execute certain functions.

## Features

#### `sendMessageWithToken`
This function transfers a token across chains, deducts a fee, and emits an event. The user deposits tokens on the bridge contract in the source chain. The contract checks that the token is approved (since `sendMessageWithToken` has the `onlyApprovedToken` modifier). This function calls `_transfer`, which performs validations. Finally, it emits a `SendMessageWithTokenEvent` containing the transaction details. Afterwards, off-chain validators sign the transaction.

##### Parameters
```solidity
address token, uint256 amount, bytes calldata payload
```

#### `sendMessage`

Emits an event with a message.

##### Parameters
```solidity
bytes calldata payload
```

#### `transferToUnwrap`

This function is called by a relayer after `sendMessageWithToken` is executed, with the purpose of sending tokens to a recipient. Relayers will listen for the corresponding `SendMessageWithTokenEvent` on the source chain and call `transferToUnwrap` providing the token, recipient and amount, a salt (a unique transaction identifier, usually the source chain transaction hash), and an array of validator signatures (proofs). The contract validates the transaction and generates a EIP-712 hash, which acts as a unique identifier for each swap.

##### Parameters
```solidity
address token,
address recipient,
uint256 amount,
bytes32 salt,
SignaturePackage[] calldata proofs
```

#### `finalizeUnwrap`

Completes an unwrap transaction and releases tokens to the recipient. If the token transfer was delayed (timelocked), the unwrap must be manually finalized. The function loops through each `orderHash` and calls `_finalizeUnwrap(orderHash)`, which verifies that the transaction has not already been completed, transfers tokens from `unwrapSent` mapping to the recipient and marks the order as sent. Finally, it emits `FinalizeUnwrapEvent.`

##### Parameters
```solidity
bytes32[] calldata orderHash
```

#### `transferToSwap`
###### _(in contract BridgeEndpointWithSwap)_`

This function executes a swap before bridging tokens. If the token is burnable, the contract mints the required amount before swapping, calls `_executeSwap` to perform the swap and emits a `TransferToSwapEvent` recording the details. If it is not burnable, it saves swap details in the `swapSent` mapping and emits `SwapOrderCreated` so the swap can be finalized later. In either case, the function will validate token and relayer permissions and generate a unique EIP-712 hash to identify the swap.

##### Parameters
```solidity
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
This function is used when a token is not burnable, leading `transferToSwap` to store the swap order instead of executing it immediately. It ensures input arrays are valid, and it loops through each `orderHash` and calls `_finalizeSwap()`.

##### Parameters
```solidity
bytes32[] calldata orderHashes,
bytes[] calldata swapPayloads
```

### Governance Features

#### `setTimeLock`

Updates the contract managing timelocks.

##### Parameters
```solidity
address _timeLock
```
#### `setTimeLockThreshold`

Sets the global timelock threshold.

##### Parameters
```solidity
uint256 _timeLockThreshold
```

#### `setTimeLockThresholdByToken`

Sets a custom timelock threshold per token.

##### Parameters
```solidity
address token, uint256 _timeLockThreshold
```

#### `addAllowList`

Adds an address to the allowlist, granting it permission to perform specific contract actions.

#### `removeAllowedList`

Removes an address from the allowlist, revoking its access.

##### Parameters
```solidity
address account
```

#### `pause`
Pauses the contract, preventing token transfers until the contract is unpaused.

#### `unpause`
Resumes contract operations after a pause, allowing bridging and transfers again.

### Read-Only Functions

#### `onAllowList`

Returns `true` if the provided address is on the allowlist, which means it has permission to use the contract’s functions.

#### `offAllowList`

Returns `true` if the provided address is not on the allowlist. 

##### Parameters
```solidity
address account
```

### Relevant Internal Functions

#### `_transfer`

Transfers a token from the sender, deducts fees, and sends the remainder to the bridge.

##### Parameters
```solidity
address token, uint256 amount
```

#### `_validateOrder`

Verifies if an order is legitimate by checking validator signatures.

##### Parameters
```solidity
bytes32 orderHash, SignaturePackage[] calldata proofs
```

#### `_finalizeUnwrap`

Completes an unwrap transaction by transferring tokens to the recipient.

##### Parameters
```solidity
(bytes32 orderHash)
```

#### `_finalizeSwap`
###### _(in contract BridgeEndpointWithSwap)_`

This function is called by `finalizeSwap` to retrieve a stored swap  order and to execute the swap. It checks if the order exists and has not been executed, transfers `amountIn` tokens from the sender to the `BridgeEndpointWithSwap` contract and calls `_executeSwap` to perform the swap. Finally, it marks the order as sent and emits a `SwapOrderFinalized` event.

##### Parameters
```solidity
bytes32 orderHash, bytes memory swapPayload
```

#### `_executeSwap`
###### _(in contract BridgeEndpointWithSwap)_`

This function approves `swapExecutor` to spend `tokenIn` and calls its `executeSwap()` function to attempt the swap. If the swap succeeds, it either burns the swapped tokens or prepares them for transfer. To transfer the tokens, `tokenOut` is sent to `pegInAddress` for bridging and a `SendMessageWithTokenEvent` is emitted. If the swap fails, the error is logged via `SwapExecutorError`, approvals are revoked and a `SendMessageWithTokenEvent` with `bridgePayloadFailure` is emitted. In either case, the function will burn `tokenIn` tokens if applicable. 

##### Parameters
```solidity
address tokenIn,
address tokenOut,
address target,
uint256 amountIn,
uint256 amountOutMin,
bytes memory swapPayload,
bytes memory bridgePayloadSuccess,
bytes memory bridgePayloadFailure
```

## Events

#### `SendMessageEvent`

Emitted when a user sends a message without transferring tokens. 

##### Parameters
```solidity
address indexed from, uint256 value, bytes payload 
```

#### `SendMessageWithTokenEvent`

Emitted when a user executes a transaction.

##### Parameters
```solidity
address indexed from,
address indexed token,
uint256 amount,
uint256 fee,
bytes payload
```

#### `TransferToUnwrapEvent`

Emitted when an order is created to unwrap tokens. This event is emitted when a transaction is validated and tokens have to be transfered to a recipient.

##### Parameters
```solidity
bytes32 orderHash,
bytes32 salt,
address indexed recipient,
address indexed token,
uint256 amount
```

#### `FinalizeUnwrapEvent`

Emitted when an unwrap order is finalized and tokens are successfully transferred to the recipient.

##### Parameters
```solidity
bytes32 indexed orderHash
```

#### `SetTimelockEvent`

Emitted when `timeLock` is updated by the contract owner.

##### Parameters
```solidity
address timeLock
```

#### `SetTimeLockThresholdEvent`

Emitted when the global time lock threshold is updated.

##### Parameters
```solidity
uint256 timeLockThreshold
```

#### `SetTimeLockThresholdByTokenEvent`

Emitted when the time lock threshold for a specific token is updated.

##### Parameters
```solidity
address token,
uint256 timeLockThreshold
```

#### `SwapExecutorError`
###### _(only present in BridgeEndpointWithSwap)_

Emitted when a swap operation fails during the bridge transfer.

##### Parameters
```solidity
address indexed target, bytes reason
```

#### `SwapOrderCreated`
###### _(only present in BridgeEndpointWithSwap)_

Emitted when a new swap order is created for non-burnable tokens. It logs the creation of swap orders, helping track orders before execution.

##### Parameters
```solidity
bytes32 indexed orderHash,
address indexed target,
address indexed tokenIn,
address tokenOut,
uint256 amountIn,
uint256 amountOutMin,
bytes bridgePayloadSuccess,
bytes bridgePayloadFailure
```

#### `SwapOrderFinalized`
###### _(only present in BridgeEndpointWithSwap)_

Emitted when a swap order is executed and finalized, confirming a swap has been processed.

##### Parameters
```solidity
bytes32 indexed orderHash,
address indexed executor,
uint256 amountOut,
bool success
```

#### `TransferToSwapEvent`
###### _(only present in BridgeEndpointWithSwap)_

Emitted when a token transfer and swap operation is executed.

##### Parameters
```solidity
bytes32 orderHash,
address target,
bytes swapPayload,
address tokenIn,
address tokenOut,
uint256 amountIn,
uint256 amountOut,
bool success
```

## Contract Calls (Interactions)

- `BridgeRegistry`: this contract is called to process orders and to manage validator roles and fees. It acts as the central registry for approved tokens, relayers and validators. 
- `ITimeLock`: provides the implementation for timelocked transactions. 
- `IBurnable`: handles the wrapping/unwrapping process in a bridge by burning tokens on one chain and minting them on another.
- `SwapExecutor`: this contract is called by `BridgeEndpointWithSwap` to execute swaps with external liquidity aggregators during bridging.