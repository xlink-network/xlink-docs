# BridgeEndpoint
- Location: `xlink/packages/contracts/bridge-solidity/contracts`
- Deployed contracts: See [Ethereum Contract Addresses](https://docs.xlink.network/xlink-network/readme/ethereum-contract-addresses).

This technical document provides a detailed overview of the Bridge Endpoint in EVM-compatible blockchains. The Bridge Endpoint facilitates communication between two blockchain networks by acting as the entry and exit point for assets moving along the Cross Chain Bridge. It passes messages between chains in the form of events, triggers contract calls, processes token transfers and validates and executes the unwrapping of tokens. `BridgeEndpointWithSwap` extends `BridgeEndpoint` and implements the necessary features to source liquidity from external aggregators.

This Bridge Endpoint functionality is implemented and distributed across the following contracts:

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

The address in which tokens will be stored before they are bridged out of the EVM-compatible blockchain. The user calls [`sendMessageWithToken`](#sendmessagewithtoken), which deducts a fee and transfers the remaining non-burnable tokens to `pegInAddress`.

### `timeLock`

| Data     | Type   |
| -------- | ------ |
| Variable | `ITimeLock` |

Manages locked transactions that require a delay before execution. Tokens amounts that exceed the threshold are not immediately sent to the user. Instead, the `timeLock` contract holds them until the waiting period expires. At that point, the `timeLock` owner can fulfill the order, completing the cross-chain transfer.

### `timeLockThreshold`

| Data     | Type   |
| -------- | ------ |
| Variable | `uint256` |

The global minimum token amount that triggers a timelock. By default, the value is set to `0`.

### `timeLockThresholdByToken`
| Data     | Type   |
| -------- | ------ |
| Variable | `mapping(address => uint256)` |

Optional custom timelock thresholds for different tokens. The timelock will be triggered when a token amount exceeds the custom threshold, if applicable. If no custom threshold is set, the token amount will need to exceed the global threshold set in [`timeLockThreshold`](#timelockthreshold).

### `unwrapSent`
| Data     | Type   |
| -------- | ------ |
| Variable | `mapping(bytes32 => OrderPackage)` |

Stores unwrap orders that need to be finalized. It stores a mapping of [`OrderPackage`](#orderpackage) structs, which contain a flag indicating whether the unwrap has been completed. `bytes32` is a unique hash of the struct parameters, as calculated by [`transferToUnwrap`](#transfertounwrap).

### `swapExecutor`
###### _(only present in BridgeEndpointWithSwap)_

| Data     | Type   |
| -------- | ------ |
| Variable | `SwapExecutor` |

Holds a reference to `SwapExecutor`, which is the contract that executes swaps.

### `swapSent`
###### _(only present in BridgeEndpointWithSwap)_

| Data     | Type   |
| -------- | ------ |
| Variable | `mapping(bytes32 => SwapOrderPackage)` |

A mapping of [`SwapOrderPackage`](#swaporderpackage) structs that contains details of swap orders.`bytes32` is a unique hash of the swap parameters, as calculated by [`transferToSwap`](#transfertoswap).

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

Contains signatures from validators to prove an order is legitimate. It is used for verifying cross-chain transfers.

```solidity
struct SignaturePackage {
  bytes32 orderHash;
  address signer;
  bytes signature;
}
```

#### `SwapOrderPackage`
###### _(only present in BridgeEndpointWithSwap)_

A struct that stores details of a swap order.

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
- `notWatchlist(recipient)`: prevents transfers to watchlisted addresses.
- `nonReentrant`: protects against reentrancy attacks.
- `onlyAllowlisted`: ensures only allowed addresses can execute certain functions.

## Features

#### `sendMessageWithToken`

This function is called to initiate the peg-out process from an EVM-compatible blockchain onto other chains, such as Stacks or Bitcoin. The user deposits tokens in the bridge contract, which are burned or locked, depending on the token. The contract checks that the token is approved, since [`sendMessageWithToken`](#sendmessagewithtoken) has the `onlyApprovedToken` modifier. The function calls [`_transfer`](#_transfer), which performs validations. Finally, the function emits a [`SendMessageWithTokenEvent`](#sendmessagewithtokenevent) containing the transaction details, which will be reviewed by validators. Once they verify that the tokens were deposited in the `BridgeEndpoint` contract, the validators will sign the order and relayers will submit it in the destination chain.   

##### Parameters
```solidity
address token,
uint256 amount, 
bytes calldata payload
```

#### `sendMessage`

Emits an event with a message.

##### Parameters
```solidity
bytes calldata payload
```

#### `transferToUnwrap`

This function is called by a relayer when a user initiates a token transfer from another blockchain. The originating order may come from an EVM chain (via [`sendMessageWithToken`](#sendmessagewithtoken)), or from a non-EVM chain like Stacks, Bitcoin or Solana. Relayers listen for the corresponding event on the source chain and call [`transferToUnwrap`](#transfertounwrap) on the destination chain, supplying the recipient, token, amount, a salt (usually the source chain transaction), and an array of validator signatures (proofs). The contract verifies these proofs and generates an EIP-712-compliant hash, which acts as a unique identifier for each order.

##### Parameters
```solidity
address token,
address recipient,
uint256 amount,
bytes32 salt,
SignaturePackage[] calldata proofs
```

#### `finalizeUnwrap`

This function completes a pending unwrap order for non-burnable tokens. It is called by a hot wallet address once the timeLock period expires, which transfers the tokens to the recipients and finalizes peg-in orders. It loops through each `orderHash` and calls [`_finalizeUnwrap()`](#_finalizeunwrap), which verifies that the order has not already been completed and transfers the token and amount stored in [`unwrapSent`](#unwrapsent) to the recipient. The order is then marked as completed, and the [`FinalizeUnwrapEvent`](#finalizeunwrapevent) is emitted.

##### Parameters
```solidity
bytes32[] calldata orderHash
```

#### `transferToSwap`
###### _(in contract BridgeEndpointWithSwap)_`

This function executes a swap before bridging tokens. If the token is burnable, the contract mints the required amount before swapping, calls [`_executeSwap`](#_executeswap) to perform the swap and emits a [`TransferToSwapEvent`](#transfertoswapevent) recording the details. If it is not burnable, it saves swap details in the [`swapSent`](#swapsent) mapping and emits [`SwapOrderCreated`](#swapordercreated) so the swap can be finalized later. In either case, the function will validate token and relayer permissions and generate a unique EIP-712 hash to identify the swap.

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
This function is used when a token is not burnable, leading [`transferToSwap`](#transfertoswap) to store the swap order instead of executing it immediately. It ensures input arrays are valid, and it loops through each `orderHash` and calls [`_finalizeSwap()`](#_finalizeswap).

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
address token, 
uint256 _timeLockThreshold
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

#### `offAllowList`

Returns `true` if the provided address is not on the allowlist. 

##### Parameters
```solidity
address account
```

#### `onAllowList`

Returns `true` if the provided address is on the allowlist, which means it has permission to use the contract’s functions.

### Relevant Internal Functions

#### `_transfer`

This internal function is responsible for processing token transfers when a user sends tokens into the bridge. It is called from [`sendMessageWithToken`](#sendmessagewithtoken) and performs validations, calculates and deducts fees, and sends the correct amount of tokens to the receiver. This function ensures the transfer amount is within allowed limits and that it is large enough to cover the minimum fee. If the token is burnable, it burns the amount minus the fee. Otherwise, it transfers the same amount to the `pegInAddress`. In either case, the fee is sent to the `BridgeRegistry` contract.

##### Parameters
```solidity
address token, 
uint256 amount
```

#### `_validateOrder`

Verifies if an order is legitimate by checking validator signatures.

##### Parameters
```solidity
bytes32 orderHash, 
SignaturePackage[] calldata proofs
```

#### `_finalizeUnwrap`

Completes an unwrap transaction by transferring tokens to the recipient.

##### Parameters
```solidity
bytes32 orderHash
```

#### `_finalizeSwap`
###### _(in contract BridgeEndpointWithSwap)_`

This function is called by [`finalizeSwap`](#finalizeswap) to retrieve a stored swap order and to execute the swap. It checks if the order exists and has not been executed, transfers `amountIn` tokens from the sender to the `BridgeEndpointWithSwap` contract and calls [`_executeSwap`](#_executeswap) to perform the swap. Finally, it marks the order as sent and emits a [`SwapOrderFinalized`](#swaporderfinalized) event.

##### Parameters
```solidity
bytes32 orderHash, 
bytes memory swapPayload
```

#### `_executeSwap`
###### _(in contract BridgeEndpointWithSwap)_`

This function approves `swapExecutor` to spend `tokenIn` and calls its `executeSwap()` function to attempt the swap. If the swap succeeds, it either burns the swapped tokens or prepares them for transfer. To transfer the tokens, `tokenOut` is sent to [`pegInAddress`](#peginaddress) for bridging and a [`SendMessageWithTokenEvent`](#sendmessagewithtokenevent) is emitted. If the swap fails, the error is logged via `SwapExecutorError`, approvals are revoked and a [`SendMessageWithTokenEvent`](#sendmessagewithtokenevent) with `bridgePayloadFailure` is emitted. In either case, the function will burn `tokenIn` tokens if applicable. 

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
address indexed from, 
uint256 value, 
bytes payload 
```

#### `SendMessageWithTokenEvent`

Emitted when a user initiates a peg-out order.

##### Parameters
```solidity
address indexed from,
address indexed token,
uint256 amount,
uint256 fee,
bytes payload
```

#### `TransferToUnwrapEvent`

Emitted when an order is created to unwrap tokens. This event is emitted when an order is validated and tokens have to be transfered to a recipient.

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

Emitted when [`timeLock`](#timelock) is updated by the contract owner.

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
address indexed target, 
bytes reason
```

#### `SwapOrderCreated`
###### _(only present in BridgeEndpointWithSwap)_

Emitted when a new swap order is created for non-burnable tokens. It logs the creation of swap orders, helping to track them before execution.

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
- `ITimeLock`: the `ITimeLock` interface is utilized to interact with the [`timeLock`](#timelock) contract by calling the `createAgreement` function when bridged amounts exceeds the threshold.
- `IBurnable`: this interface is used for burnable tokens to enable mint and burn operations.
- `ERC20`:  all approved tokens within the bridge must implement the `ERC20Fixed` standard, which is an XLink's custom standard that extends ERC-20 to handle fixed precision. Token contract interactions occur in both peg-in and peg-out operations for non-burnable tokens, using the `transferFromFixed`, `transferFixed`, and `increaseAllowanceFixed` functions.
- `SwapExecutor`: this contract is called by `BridgeEndpointWithSwap` to execute swaps with external liquidity aggregators during bridging.