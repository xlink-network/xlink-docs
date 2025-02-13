# btc-peg-in-v2-07a-agg.clar

- Location: `xlink/packages/contracts/bridge-stacks/contracts/btc-peg-in-v2-07a-agg.clar`
- [Deployed contract]()

This technical document provides a detailed overview of the contract responsible for managing the peg-in process when using non-ALEX liquidity aggregators. This allows users to trade tokens even when the pair isn't available in the ALEX AMM or is lacking liquidity. This contract enables the transfer of BTC from the Bitcoin network to the Stacks network in the form of aBTC. The contract's primary functionality is implemented through the `finalize-peg-in-agg` function and its associated private functions. Let's review this core operation.

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

The percentage fee charged for peg-in transactions. By default, this value is `u0`.

### `peg-in-min-fee`
| Data     | Type   |
| -------- | ------ |
| Variable | `uint` |

The minimum fee required for a peg-in transaction, regardless of the transaction amount. For each transaction, the fee is determined by taking the maximum value between the specified fee amount and the `peg-in-min-fee`. By default, this value is `u0`.

### `btc-peg-outfee`
| Data     | Type   |
| -------- | ------ |
| Variable | `uint` |

The percentage fee charged for peg-in transactions. By default, this value is `u0`.

### `btc-peg-out-min-fee`
| Data     | Type   |
| -------- | ------ |
| Variable | `uint` |

The minimum fee required for a peg-out transaction, regardless of the transaction amount. For each transaction, the fee is determined by taking the maximum value between the specified fee amount and the `peg-out-min-fee`. By default, this value is `u0`.

## Features

### Aggregator Peg-In Feature

#### `finalize-peg-in-agg`

This function manages the basic aggregator peg-in process. It checks if the provided Bitcoin transaction has been mined. It also calculates the appropriate fees for the peg-in operation. The function verifies that the transaction meets the necessary conditions, including checking that the peg-in address is approved, before finalizing it by registering it in the `.cross-peg-out-v2-01-agg` contract to handle cross-chain operations. Finally, it mints bridged BTC tokens for the recipient address specified in the Bitcoin transaction, processes fees, and logs the transaction details.

##### Parameters
```lisp
(tx (buff 32768))
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(output-idx uint) 
(reveal-tx { tx: (buff 32768), order-idx: uint })
(reveal-block { header: (buff 80), height: uint })
(reveal-proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
```

### Governance Features

#### `set-fee-to-address`
This standard protocol function checks whether a caller (`tx-sender`) is the DAO executor or an authorized extension, delegating the extensions check to the `executor-dao` contract.

##### Parameters
```lisp
(new-fee-to-address principal)
```

#### `pause-peg-in`
A public function, governed through the `is-dao-or-extension`, that can change the contract's operational status.

##### Parameters
```lisp
(paused bool)
```

#### `set-peg-in-fee`
This feature sets the fee for the peg-in operation as a percentage of the transaction amount to be deducted.

##### Parameters
```lisp
(fee uint)
```

#### `set-peg-in-min-fee`
This feature establishes the minimum fee required for a peg-in transaction.

#### `set-btc-peg-out-fee`
This feature sets the fee for the peg-out operation as a percentage of the transaction amount to be deducted.

#### `set-btc-peg-out-min-fee`
This feature establishes the minimum fee required for a peg-out transaction.

##### Parameters
```lisp
(fee uint)
```

### Supporting Features
The following functions are tools to assist the off-chain activities.
1. Construct and destruct helpers (`destruct-principal`, `construct-principal`).
2. Order creation helper (`create-order-agg-or-fail`).
3. Decoding helpers (`decode-order-agg-or-fail`, `decode-order-agg-from-reveal-tx-or-fail`).

### Getters

#### `get-peg-in-fee`
#### `get-peg-in-min-fee`
#### `get-btc-peg-out-fee`
#### `get-btc-peg-out-min-fee`
#### `get-fee-to-address`
#### `get-peg-in-sent-or-default`
##### Parameters
```lisp
(tx (buff 32768))
(output uint)
```

#### `get-txid`
##### Parameters
```lisp
(tx (buff 32768))
```

## Contract Calls (Interactions)

- `executor-dao`: calls are made to verify whether a certain contract-caller is designated as an extension.
- `btc-bridge-registry-v2-01`: this contract is called to validate peg-in addresses, check and set transaction statuses, and order details during peg-in operations.
- `bridge-common-v2-02`: this contract is called to extract and validate Bitcoin transaction details, decode routing and order information during peg-in operations.
- `token-abtc`: this contract handles the management of aBTC (Bridged BTC) tokens, representing BTC on the Stacks network. It is called to mint and transfer aBTC during peg-in operations.
- `cross-peg-out-v2-01-agg`: this contract is called to finalize the transaction and retrieve the transaction details necessary for the peg-out process.
- `clarity-bitcoin-v1-07`: calls are made to validate transaction details.

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