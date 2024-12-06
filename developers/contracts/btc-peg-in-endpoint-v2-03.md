# btc-peg-in-endpoint-v2-03.clar

- Location: `xlink/packages/contracts/bridge-stacks/contracts/btc-peg-in-endpoint-v2-03.clar`

This technical document provides a detailed overview of the contract responsible for managing the peg-in process, enabling the transfer of BTC from the Bitcoin network to the Stacks network. In this process, BTC is represented as bridged tokens on Stacks (aBTC). The contract's primary functionality is implemented through a series of public functions. Let's review this core operation.

## Storage
### `fee-to-address`
| Data     | Type   |
| -------- | ------ |
| Variable | `principal` |

The address where the fees collected from peg-in operations are transferred. By default, this address is set to the executor-dao responsible for governance.

### `peg-in-paused`
| Data     | Type   |
| -------- | ------ |
| Variable | `bool` |

A flag that indicates whether the peg-in process is active. If set to true, all peg-in operations are paused, preventing any new transactions. The contract is deployed in a paused state by default.

### `peg-in-fee`
| Data     | Type   |
| -------- | ------ |
| Variable | `uint` |

The percentage fee charged for peg-in transactions. By default, this value is u0.

### `peg-in-min-fee`
| Data     | Type   |
| -------- | ------ |
| Variable | `uint` |

The minimum fee required for a peg-in transaction, regardless of the transaction amount. For each transaction, the fee is determined by taking the maximum value between the specified fee amount and the `peg-in-min-fee`. By default, this value is `u0`.

## Features

### Peg-in features

#### `finalize-peg-in-0`
This function manages the basic peg-in process for transferring BTC to Stacks. It validates the provided Bitcoin transaction using a proof that confirms it has been mined and calculates the appropriate fees for the operation. The function verifies that the transaction meets the necessary conditions, including checking that the peg-in address is approved, before finalizing it by registering it in the `.btc-bridge-registry-v2-02` contract. Finally, it mints bridged BTC tokens for the recipient address specified in the Bitcoin transaction, processes fees, and logs the transaction details.

##### Parameters
```lisp
(tx (buff 32768))
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(output-idx uint) 
(order-idx uint)
```

#### `finalize-peg-in-cross`
Building upon the checks performed in `finalize-peg-in-0`, this function extends the peg-in process to support cross-chain routing. After validating the Bitcoin transaction, it performs additional checks involving a "reveal transaction", which specifies the token and chain-id for the destination chain. The function calculates fees and verifies the token trait of the assets being transferred against the `.btc-bridge-registry-v2-02` contract to ensure it is approved and registers the transaction status. Once validated, the function mints bridged BTC tokens and sends them to the recipient specified in the transaction details via the `.cross-router-v2-02 contract`.

##### Parameters
```lisp
(tx (buff 32768))
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(output-idx uint)
(reveal-tx { tx: (buff 32768), order-idx: uint })
(token-trait <ft-trait>)
```

#### `finalize-peg-in-cross-swap`
This function builds on `finalize-peg-in-cross` by adding token swapping capabilities during the peg-in process. In addition to the verifications performed in `finalize-peg-in-cross`, it ensures that routing configurations (e.g., minimum output amounts, token paths) are valid and meets all conditions. It interacts with `.btc-bridge-registry-v2-02` contract to register transaction statuses, uses `.cross-router-v2-02` to handle the cross operation, and to perform token swaps through the `.amm-pool-v2-01` contract. The function mints bridged BTC tokens and swaps them according to routing instructions.

##### Parameters
```lisp
(tx (buff 32768))
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(output-idx uint) 
(reveal-tx { tx: (buff 32768), order-idx: uint })
(routing-traits (list 5 <ft-trait>))
```

#### `finalize-peg-in-launchpad`
This function is tailored for peg-ins associated with launchpad projects on Stacks. It uses `.btc-bridge-registry-v2-02` to validate and register transaction details while ensuring compatibility with the project’s parameters. These parameters include fields such as `user`, `launch-id`, and `payment-token-trait`, which define the specifics of the launchpad operation. 
The function first mints bridged BTC tokens for the user and then registers the operation in the `.alex-launchpad-v2-01` contract. This registration involves transferring the minted bridged tokens assets to the launchpad contract on behalf of the user, where they are associated with the launchpad project. 
In case of any error, it invokes the internal [refund]() function and logs the issue.

##### Parameters
```lisp
(tx (buff 32768))
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(output-idx uint)
(order-idx uint)
```

### Governance features
#### `is-dao-or-extension`
This standard protocol function checks whether a caller (`tx-sender`) is the DAO executor or an authorized extension, delegating the extensions check to the `executor-dao` contract.

#### `is-peg-in-paused`
A read-only function that checks the operational status of the contract.

#### `pause-peg-in`
A public function, governed through the `is-dao-or-extension`, that can change the contract's operational status.

##### Parameters
```lisp
(paused bool)
```

#### `set-fee-to-address`
This feature establishes the address where the fees collected from peg-in operations are transferred.

##### Parameters
```lisp
(new-fee-to-address principal)
```

#### `set-peg-in-fee`
This feature sets the fee for the peg-in operation as a percentage of the transaction amount to be deducted.

##### Parameters
```lisp
(fee uint)
```

#### `set-peg-in-min-fee`
This feature establishes the minimum fee required for a peg-in transaction.

##### Parameters
```lisp
(fee uint)
```

### Supporting features
The following functions are tools to assist the off-chain activities.
1. Construct and destruct helpers (`destruct-principal`, `construct-principal`).
2. Order creation helpers (`create-order-0-or-fail`, `create-order-cross-or-fail`, `create-order-cross-swap-or-fail`, `create-order-launchpad-or-fail`).
3. Decoding helpers (`decode-order-cross-from-reveal-tx-or-fail`, `decode-order-cross-swap-from-reveal-tx-or-fail`).


### Relevant internal functions
- `refund`


### Getters

#### `get-peg-in-fee`
#### `get-peg-in-min-fee`
#### `get-fee-to-address`
#### `get-peg-in-sent-or-default`
##### Parameters
```lisp
(tx (buff 32768))
(output uint)
```
#### `get-approved-wrapped-or-default`
Checks if the given token is approved as a wrapped token by consulting the `approved-wrapped` map in the `.btc-bridge-registry-v2-02` contract. If the token is not found in the map, it returns false as the default value.
##### Parameters
```lisp
(token principal)
```
#### `get-txid`
##### Parameters
```lisp
(tx (buff 32768))
```

## Contract calls (interactions)
- `executor-dao` Calls are made to verify whether a certain contract-caller is designated as an extension.
- `btc-bridge-registry-v2-02` This contract is called to validate peg-in addresses, check and set transaction statuses, and order details during peg-in operations.
- `bridge-common-v2-02` This contract is called to extract and validate Bitcoin transaction details, decode routing and order information during peg-in operations.
- `token-abtc` This contract handles the management of aBTC (Bridged BTC) tokens, representing BTC on the Stacks network. It is called to mint and transfer aBTC during peg-in operations.
- `cross-router-v2-02` This contract is called to route tokens and execute cross-chain transfers during advanced peg-in operations, such as cross and cross-swap transactions.
- `alex-launchpad-v2-01` This contract is called to register and validate peg-in operations associated with launchpad projects on the Stacks network.
- `clarity-bitcoin-v1-07` This contract is called to retrieve the transaction ID of native SegWit Bitcoin transactions by excluding witness data.
- `btc-peg-out-endpoint-v2-01` This contract is called to manage refunds during peg-in failures to transfer BTC back to users.

## Errors

| Error Name       | Value         |
| ---------------- | ------------- |
| `err-unauthorised` | `(err u1000)` |
| `err-paused`    | `(err u1001)` |
| `err-peg-in-address-not-found` | `(err u1002)` |
| `err-invalid-amount`    | `(err u1003)` |
| `err-invalid-tx`   | `(err u1004)` |
| `err-already-sent`   | `(err u1005)` |
| `err-bitcoin-tx-not-mined`    | `(err u1011)` |
| `err-invalid-input`    | `(err u1012)` |
| `err-token-mismatch`    | `(err u1015)` |
| `err-slippage` | `(err u1016)` |
| `err-not-in-whitelist`    | `(err u1017)` |
| `err-invalid-routing`    | `(err u1018)` |
| `err-commit-tx-mismatch`    | `(err u1019)` |
| `err-invalid-token`    | `(err u1020)` |

<!-- Documentation Contract Template v0.1.0 -->