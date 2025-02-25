# BridgeEndpointWithSwap.sol

- Location: `packages/contracts/bridge-solidity/contracts/BridgeEndpointWithSwap.sol`
- [Deployed contract]()

This technical document provides a detailed overview of the contract responsible for transferring a swap between two tokens to **IceCreamSwap**, a non-ALEX liquidity aggregator. The contract burns or transfers the base tokens and sends the necessary information to complete the swap to IceCreamSwap. The main logic is implemented in the `transferToSwap` function. Lets review its core functionality.

## Storage

### `_owner`
| Data     | Type   |
| -------- | ------ |
| Variable | `address` |

This address corresponds to the owner of the contract.

### `name`
| Data     | Type   |
| -------- | ------ |
| Variable | `string memory` |

This variable stores the name of the contract.

### `version`
| Data     | Type   |
| -------- | ------ |
| Variable | `string memory` |

This variable stores the version of the contract that is deployed.

### `_registry`
| Data     | Type   |
| -------- | ------ |
| Variable | `address` |

This address corresponds to the `cross-bridge-registry-v2-01` contract, that stores information on all transactions.

### `_pegInAddress`
| Data     | Type   |
| -------- | ------ |
| Variable | `address` |

This parameter represents the address of the peg-in contract.

### `_timeLock`
| Data     | Type   |
| -------- | ------ |
| Variable | `address` |

This variable stores the address of the time lock contract.

## Features

### Swap Features

#### `transferToSwap`

This function implements the main logic to swap tokens through the bridge endpoint. calculates the `orderHash` taking `salt` as one of the parameters for the calculation. It also validates the order. It calculates `amountOut`, which seems like the amount sent out after the transfer. it either burns `tokenOut` or transfers using `pegInAddress`. It emits two events: `TransferToSwapEvent` and `SendMessageWithTokenEvent`. If unsuccessful, it burns `tokenIn`.

##### Parameters
```solidity
(
    address target,
    address tokenIn,
    address tokenOut,
    uint256 amountIn,
    uint256 amountOutMin,
    bytes calldata swapPayload,
    bytes calldata bridgePayload,
    bytes32 salt,
    SignaturePackage[] calldata proofs
)
```

### Governance Features

#### `setTimeLock`
This function is called in the constructor of the contract to ensure that _timeLock is set to the appropriate value.

##### Parameters
```solidity
(_timeLock)
```

## Events

#### `TransferToSwapEvent`
This event is emitted when a token swap is performed within the bridge endpoint. `tokenIn` represents the base token in the swap, whereas `tokenOut` represents the token it is being bridged to. `amountIn` and `amountOut` correspond to the amount of each token. The `swapPayload` parameter is a data payload that is necessary for the swap to execute. `success` indicates whether the swap could be performed or not.

##### Parameters
```solidity
(
    bytes32 orderHash,
    bytes32 salt,
    address target,
    bytes swapPayload,
    address tokenIn,
    address tokenOut,
    uint256 amountIn,
    uint256 amountOut,
    bool success
)
```

#### `SendMessageWithTokenEvent`
When a token is burnt or transferred to another blockchain, this event is emitted to log the transaction corresponding to that token. Similarly, when a token is minted in the target blockchain after a swap is performed, this event will also be emited to record that operation.

##### Parameters
```solidity
(
    address indexed from,
    address indexed token,
    uint256 amount,
    uint256 fee,
    bytes payload
)
```

