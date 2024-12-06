# meta-peg-in-endpoint-v2-02.clar

- Location: `xlink/packages/contracts/bridge-stacks/contracts/meta-peg-in-endpoint-v2-02.clar`
- [Deployed contract]()

This technical document provides a comprehensive overview of the contract responsible for the peg-in (bridging) of Bitcoin's BRC-20 assets to the Stacks network. BRC-20 is a token standard on Bitcoin's metaprotocol layer, enabling fungible assets similar to Ethereum's ERC-20. The primary operation of the contract is to facilitate the peg-in process, which involves transferring these BRC-20 assets from Bitcoin to the Stacks blockchain as a specific token. This functionality is implemented through several public functions, each designed to operate with slight variations tailored to specific use cases. Let's explore them.

## Storage

### `paused`

| Data     | Type   |
| -------- | ------ |
| Variable | `bool` |

This data variable serves as a flag to control the operational status of the contract within the system. When set to 'paused,' it blocks all meta peg-in transactions. By default, the contract is deployed in 'paused' mode. Refer to the [`pause`](meta-peg-in-endpoint-v2-02.md#pause) governance feature to change this status.

### `fee-to-address`

| Data     | Type   |
| -------- | ------ |
| Variable | `principal` |

This is the address to which the fee in Bridged BTC tokens (`token-abtc`) is transferred. By default, this address is the `tx-sender` who deployed the contract.

### `peg-in-fee`

| Data     | Type   |
| -------- | ------ |
| Variable | `uint` |

This value represents the minimum fee to be deducted for the peg-in operation, expressed in fixed BTC (Bitcoin's base unit: satoshis). Fee amounts below this limit will be rejected with the error `err-invalid-amount`. By default, this value is set to `u0`.

### Relevant constants

#### `burn-height-start`

| Type   | Value |
| ------ | ----- |
| `uint` | `burn-block-height` |

This constant denotes the block height at which the contract was deployed on the underlying burn blockchain. It serves to restrict operations to only include transactions minted from this block onward.

## Features

### Peg-in features

#### `finalize-peg-in-on-index`

In its simplest form, the peg-in operation is a function that processes a Bitcoin transaction involving a token transfer on that blockchain (e.g., BRC-20), effectively wrapping the foreign token into a local Stacks token. This token represents the asset from which net funds will be transferred to the designated order address. The function requires proofs and validation data, provided by other protocol participants, to confirm the authenticity of the external blockchain transaction. Additionally, the function call may include information about a fee charged in the origin blockchain's base-coin (e.g., BTC), which will be minted in favor of the sender in Stacks as the special Bridged BTC token (`token-abtc`). For the transaction's assets, both the destination (`to`) address and the token must be registered in the auxiliary contract `meta-bridge-registry-v2-03`. Similarly, the address receiving the fees must be previously approved in the contract `btc-peg-in-endpoint-v2-03` (see interaction below).

##### Parameters

```lisp
(tx {
      bitcoin-tx: (buff 32768),
      output: uint,
      tick: (string-utf8 256),
      amt: uint,
      from: (buff 128),
      to: (buff 128),
      from-bal: uint,
      to-bal: uint,
      decimals: uint
    })
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(signature-packs (list 10 { signer: principal, tx-hash: (buff 32), signature: (buff 65) }))
(order-idx uint)
(fee-idx (optional uint))
(token-trait <ft-trait>)
```

#### `finalize-drop-peg-in-on-index`

This variation of the peg-in feature involves obtaining the order address for asset transfer from the reveal transaction, which is received as an argument.

##### Parameters

```lisp
(tx {
      bitcoin-tx: (buff 32768),
      output: uint,
      tick: (string-utf8 256),
      amt: uint,
      from: (buff 128),
      to: (buff 128),
      from-bal: uint,
      to-bal: uint,
      decimals: uint
    })
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(signature-packs (list 10 { signer: principal, tx-hash: (buff 32), signature: (buff 65) }))
(reveal-tx { tx: (buff 32768), order-idx: uint })
(fee-idx (optional uint))
(token-trait <ft-trait>)
```

#### `finalize-peg-in-cross-on-index`

This peg-in-cross feature processes the order recipient specified in the reveal transaction, which is provided as an argument. In this scenario, the recipient may be on a different blockchain. To facilitate this, the contract calls the supporting contract `cross-router-v2-02`, which uses the following logic:

```
chain-id > 1000 // BRC-20
chain-id == 0 // BTC
chain-id < 1000 // EVM networks
chain-id == None // Stacks
```

As this peg-in feature is designed for Bitcoin metaprotocol tokens (e.g., BRC-20), the primary objective of this function is to facilitate 'crossing' to an EVM network. In such cases, the `cross-router-v2-02` contract will subsequently call `cross-peg-out-endpoint-v2-01` to execute the transfer. This function includes a refund feature to handle any errors that may occur during cross-transaction data validation.

##### Parameters

```lisp
(tx {
      bitcoin-tx: (buff 32768),
      output: uint,
      tick: (string-utf8 256),
      amt: uint,
      from: (buff 128),
      to: (buff 128),
      from-bal: uint,
      to-bal: uint,
      decimals: uint
    })
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(signature-packs (list 10 { signer: principal, tx-hash: (buff 32), signature: (buff 65) }))
(reveal-tx { tx: (buff 32768), order-idx: uint })
(fee-idx (optional uint))
(token-trait <ft-trait>)
```

#### `finalize-peg-in-cross-swap-on-index`

Similar to the peg-in-cross, the peg-in-cross-swap involves a cross-blockchain operation with an additional asset swapping capability. This feature is useful when the desired target token requires intermediate swap operations. The function achieves this by receiving a list of token traits involved in the operation up to the target token. Refer to the [AMM Pool documentation]() for more details. In addition to the "from" and "to" addresses, the order details in the reveal transaction include information about the swap route. This swap route is validated against the `amm-pool-v2-01` contract through the `cross-router-v2-02` contract to check pool parameters. Additionally, this function includes a refund mechanism to handle errors during cross-swap transaction validation.

##### Parameters

```lisp
(tx {
      bitcoin-tx: (buff 32768),
      output: uint,
      tick: (string-utf8 256),
      amt: uint,
      from: (buff 128),
      to: (buff 128),
      from-bal: uint,
      to-bal: uint,
      decimals: uint
    })
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(signature-packs (list 10 { signer: principal, tx-hash: (buff 32), signature: (buff 65) }))
(reveal-tx { tx: (buff 32768), order-idx: uint })
(fee-idx (optional uint))
(routing-traits (list 5 <ft-trait>))
```

#### `finalize-peg-in-add-liquidity-on-index`

This function enables a peg-in operation that injects liquidity into ALEX's AMM Trading Pool system (contract `amm-pool-v2-01`). Instead of transferring the pegged assets (`token-trait`), it initiates an operation to add to positions with the token pair `token-abtc` and the `token-trait` provided in the function arguments. This liquidity operation is recorded in the `meta-bridge-registry-v2-03`, including the order generator. It's important to note that the order generator (`from`), the factor, and the maximum target token amount (`max-dy`) are extracted from the order details in the `reveal-tx` argument. The function includes a refund feature to reimburse any difference between the net amount and the injected target token in the pool operation, as well as to handle errors in the add-liquidity transaction data validation.

##### Parameters

```lisp
(tx {
      bitcoin-tx: (buff 32768),
      output: uint,
      tick: (string-utf8 256),
      amt: uint,
      from: (buff 128),
      to: (buff 128),
      from-bal: uint,
      to-bal: uint,
      decimals: uint
    })
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(signature-packs (list 10 { signer: principal, tx-hash: (buff 32), signature: (buff 65) }))
(reveal-tx { tx: (buff 32768), order-idx: uint })
(dx-idx uint)
(fee-idx (optional uint))
(token-trait <ft-trait>)
```

#### `finalize-peg-in-remove-liquidity`

This operation performs the inverse of the add liquidity feature, focusing on reducing positions. After removing positions from ALEX's AMM Trading Pool system and recording the liquidity removed (`meta-bridge-registry-v2-03`), the function initiates two peg-out requests due to the injected liquidity being for the pair `token-abtc` + `token-trait`. For the first token of the pair, which represents the burn blockchain base-coin (BTC), the function calls `btc-peg-out-endpoint-v2-01`. For the second token, which wraps the BRC-20 assets, the function calls `meta-peg-out-endpoint-v2-03`. These two contracts manage the peg-out requests.

##### Parameters

```lisp
(commit-tx { tx: (buff 32768), fee-idx: (optional uint) })
(block { header: (buff 80), height: uint })
(proof { tx-index: uint, hashes: (list 14 (buff 32)), tree-depth: uint })
(reveal-tx { tx: (buff 32768), order-idx: uint })
(token-trait <ft-trait>)
```

### Supporting features

The following functions are tools to assist the off-chain activities.

1. Validation helpers (`validate-tx-cross`, `validate-tx-cross-swap`, `validate-tx-add-liquidity`).
2. Order creation helpers (`create-order-cross-or-fail`, `create-order-cross-swap-or-fail`, `create-order-add-liquidity-or-fail`, `create-order-remove-liquidity-or-fail`).
3. Decoding helpers (`decode-order-cross-or-fail`, `decode-order-cross-swap-or-fail`).
4. Token pair helpers (`get-pair-details-many`, `is-approved-pair`).

### Governance features

#### `is-dao-or-extension`

This standard protocol function checks whether a caller (`tx-sender`) is the DAO executor or an authorized extension, delegating the extensions check to the `executor-dao` contract.

#### `is-paused`

A read-only function that checks the operational status of the contract.

#### `pause`

A public function, governed through the `is-dao-or-extension`, that can change the contract's operational status.

##### Parameters

```lisp
(new-paused bool)
```

#### `transfer-all-to-many`

This contract feature is designed to transfer its entire balance of a specified list of tokens to a new owner. Access to this feature is restricted by the `is-dao-or-extension` function.

##### Parameters

```lisp
(new-owner principal) (token-traits (list 10 <ft-trait>))
```

#### `set-fee-to-address`

This contract feature sets the address to which the fee in Bridged BTC tokens (`token-abtc`) will be transferred.

##### Parameters

```lisp
(new-fee-to-address principal)
```

#### `set-peg-in-fee`

This feature establishes the fee to be deducted for the peg-in operation.

##### Parameters

```lisp
(fee uint)
```

### Getters

* `get-fee-to-address`
* `get-peg-in-fee`
* `get-pair-details`
* `get-pair-details-or-fail`
* `get-pair-details-many`
* `get-tick-to-pair-or-fail`
* `get-peg-in-sent-or-default`

### Relevant internal functions

* `index-tx`
* `refund`

## Contract calls (interactions)

* `executor-dao`: Calls are made to verify whether a certain contract-caller is designated as an extension.
* `meta-bridge-registry-v2-03`: This interaction is used by the peg-in contract to check and set token pair information. It also facilitates liquidity operations to set balances and manage pool operations.
* `btc-peg-in-endpoint-v2-03`: This contract is called to verify Bitcoin operations, ensuring that a given Bitcoin transaction hasn't been processed before and that the approved address is used to receive the burn blockchain base-coin.
* `btc-peg-out-endpoint-v2-01`: The `request-peg-out-0` function of this contract is used by the peg-in contract to remove liquidity and refund BTC amounts.
* `btc-bridge-registry-v2-02`: The peg-in contract calls the `set-peg-in-sent` function of this contract to register a processed Bitcoin transaction.
* `meta-peg-out-endpoint-v2-03`: The `finalize-peg-in-remove-liquidity` feature and the internal `refund` function call this contract to request a peg-out in their respective contexts.
* `cross-router-v2-02`: The peg-in cross and peg-in cross swap operations use this contract to validate and route peg-ins involving other blockchains.
* `oracle-v2-01`: This contract is called to verify that a transaction was mined and to index Bitcoin transactions.
* `clarity-bitcoin-v1-07`: The peg-in contract uses this contract to obtain legacy transaction IDs from a SegWit native transaction.
* `amm-pool-v2-01`: This contract manages all liquidity operations, including adding and reducing positions, within the peg-in features. Additionally, it acts as a helper to retrieve pool information and details.
* `token-abtc`: This is the Bridged BTC token used throughout the peg-in contract to operate with native Stacks' SIP-010 token transactions involving the burn blockchain base-coin (BTC).
* Tokens (`token-trait`) During peg-in operations, this trait is used to invoke the relevant tokens to perform the necessary transfers involved in the transaction. It is a customized version of Stacks' standard definition for Fungible Tokens (`sip-010`), with support for 8-digit fixed notation.
* `bridge-common-v2-02`: This contract provides common helper functions to all xLink contracts, including order creation, transaction decoding, and cross-chain routing validations.


## Errors

| Error Name                        | Value         |
| --------------------------------- | ------------- |
| `err-unauthorised`                | `(err u1000)` |
| `err-paused`                      | `(err u1001)` |
| `err-peg-in-address-not-found`    | `(err u1002)` |
| `err-invalid-amount`              | `(err u1003)` |
| `err-token-mismatch`              | `(err u1004)` |
| `err-invalid-tx`                  | `(err u1005)` |
| `err-already-sent`                | `(err u1006)` |
| `err-address-mismatch`            | `(err u1007)` |
| `err-request-already-revoked`     | `(err u1008)` |
| `err-request-already-finalized`   | `(err u1009)` |
| `err-revoke-grace-period`         | `(err u1010)` |
| `err-request-already-claimed`     | `(err u1011)` |
| `err-invalid-input`               | `(err u1012)` |
| `err-tx-mined-before-request`     | `(err u1013)` |
| `err-commit-tx-mismatch`          | `(err u1014)` |
| `err-invalid-burn-height`         | `(err u1003)` |
| `err-tx-mined-before-start`       | `(err u1015)` |
| `err-slippage-error`              | `(err u1016)` |
| `err-bitcoin-tx-not-mined`        | `(err u1017)` |


<!-- Documentation Contract Template v0.1.0 -->
