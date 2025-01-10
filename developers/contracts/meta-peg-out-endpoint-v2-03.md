# meta-peg-out-endpoint-v2-03.clar

- Location: `xlink/packages/contracts/bridge-stacks/contracts/meta-peg-out-endpoint-v2-03.clar`
- [Deployed contract]()

This technical document provides a detailed overview of the contract responsible for facilitating the peg-out process of tokens from the Stacks network to the burn blockchain. The target token standard is BRC-20, a protocol on Bitcoin's metaprotocol layer that supports fungible assets, inspired by Ethereum's ERC-20 standard.

Unlike the meta-peg-in contract, the meta-peg-out feature is specifically designed to support the bridging-out process. This is achieved through a series of public functions, each intended to execute sequentially, while incorporating grace periods between their execution. In the following sections, we will explore these functions in detail.

## Storage

### `paused`

| Data     | Type   |
| -------- | ------ |
| Variable | `bool` |

This data variable serves as a flag to control the operational status of the contract within the system. When set to 'paused,' it blocks all peg-out transactions. By default, the contract is deployed in 'paused' mode. Refer to the [pause](meta-peg-out-endpoint-v2-03.md#pause) governance feature to change this status.

### `fee-to-address`

| Data     | Type   |
| -------- | ------ |
| Variable | `principal` |

This variable represents the address to which fees are paid. In this contract, there are two categories of fees: peg-out fees and gas fees. Peg-out fees are transactioned with the relevant token, while gas fees are handled using the Bridged BTC token (`token-abtc`). For more details on these transactions, refer to the [finalize peg-out feature](meta-peg-out-endpoint-v2-03.md#finalize-peg-out-on-index). By default, the address assigned to receive these fees is the `tx-sender` address of the contract deployer.

### Relevant constants

#### `burn-height-start`

| Type   | Value |
| ------ | ----- |
| `uint` | `burn-block-height` |

This constant specifies the block height of the underlying burn blockchain at the time of contract deployment. It is utilized to ensure that operations within the `finalize-peg-out` function are limited to transactions minted from this block onward.

## Features

### Peg-out features

The peg-out feature is a multi-step process comprising several phases to ensure secure and orderly transactions. The process begins with requesting a peg-out to initiate the transfer of tokens out of the Stacks network. Following this request, configured grace periods allow for specific actions: users can choose to revoke their request or claim it. Once a peg-out request is claimed, the final step is to complete the transaction by finalizing the peg-out.

#### `request-peg-out`

The first step in the peg-out process is to submit a request. This operation involves the `tx-sender` (requester) initiating a peg-out request for a specified amount of a pegged-in token. Upon initiation, the net amount along with fee costs are escrowed by transferring the token and the gas fee token (`token-abtc`) to the contract. These are held until the request is either finalized or revoked.

To proceed, the request must satisfy several validations:

- The pair (token + chain-id) must be approved in the meta registry.
- The specified amount must exceed the peg-out operation fee for that particular token.
- The token peg-out operation must be active in the registry (not paused).

Once these validations are met, the operation is registered in the meta registry via the contract `meta-bridge-registry-v2-03`. The function then generates a unique identifier for the request, known as the `request-id`, ensuring traceability and reference for future operations.

##### Parameters

```lisp
(amount uint)
(peg-out-address (buff 128))
(token-trait <ft-trait>)
(the-chain-id uint)
```

#### `claim-peg-out`

Following the peg-out request, a next potential step is to claim the request. This step is critical, as it marks the request as ready for finalization. This action is executed by calling the function with the `request-id` of the previously created request, along with the `fulfilled-by` parameter, which designates the address responsible for executing the burn blockchain operation.

The claim must satisfy the following validations:

- The request must exist.
- The token pair must be approved and operational (not paused for peg-out).
- The request must not be revoked, finalized, or previously claimed.

Once these conditions are met, the final step is to register the claim in the meta registry. This registration includes the claimer, the fulfilled-by address, and the calculated grace period, during which the claimed request will be eligible for finalization.

##### Parameters

```lisp
(request-id uint)
(fulfilled-by (buff 128))
```

#### `finalize-peg-out-on-index`

Finalizing a peg-out is the final step in committing the peg-out process. This function takes in the burn blockchain transaction (Bitcoin) that corresponds to the Stacks layer operation and executes the peg-out request (`request-id`) along with all related token transfers and their corresponding fees. The finalization can be performed by either a peg-in address or a third-party address.

Key validations include:
- The request must exist.
- The token pair must be approved and operational (not paused for peg-out).
- The transaction cannot be indexed to a time before the contract deployment.
- The transaction's metaprotocol token (BRC-20) must match the requested details (`tick` and `amount`). Additionally, the `from` address must correspond to the `fulfilled-by` address, and the `to` address must match the `peg-out-address`.
- The request must not be revoked or already finalized.

The procedure is as follows, based on who finalizes it:

1) Peg-In Address
    - The net amount of the peg-in token requested is burned from the contract balance, affecting the overall total supply.
    - The peg-out fee (involved token) and gas fees (`token-abtc`) are paid to the [configured address](meta-peg-out-endpoint-v2-03.md#fee-to-address).

2) Third-Party Address
    - The net amount of the peg-in token and the burn blockchain gas fee are transferred from the contract to the claimer, so the overall involved token balance remains unaffected.
    - The peg-out fee is transferred from the contract to the [configured address](meta-peg-out-endpoint-v2-03.md#fee-to-address).

This operation involves indexing the specified transaction through the `oracle-v2-01` contract and marking the request as "finalized" in the `meta-bridge-registry-v2-03` contract.

##### Parameters

```lisp
(request-id uint)
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
(token-trait <ft-trait>)
```

#### `revoke-peg-out`

In certain scenarios, a requested peg-out may need to be revoked, allowing users to retract their request if necessary.

Key validations include:

- The grace period must have elapsed.
- The request must not have been claimed, finalized, or previously revoked.

If these conditions are met, the process continues by returning the tokens initially escrowed for the request, including both the peg-in tokens and the associated fees. These returns are made from the contract directly back to the requester. Finally, the `meta-bridge-registry-v2-03` contract is invoked to mark this request as `revoked`.

##### Parameters

```lisp
(request-id uint)
(token-trait <ft-trait>)
```

### Supporting features

The following functions are tools to assist the off-chain activities.

1. Validation helpers (`validate-peg-out`).
2. Token pair helpers (`get-pair-details-many`).

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

This contract feature allows for specifying the address to which peg-out fees (denominated in the relevant tokens) and gas fees (denominated in Bridged BTC tokens, known as `token-abtc`) will be transferred.

##### Parameters

```lisp
(new-fee-to-address principal)
```

### Getters

* `get-fee-to-address`
* `get-request`
* `get-request-many`
* `get-request-revoke-grace-period`
* `get-request-claim-grace-period`
* `get-request-or-fail`
* `get-pair-details`
* `get-pair-details-many`
* `get-pair-details-or-fail`
* `get-tick-to-pair-or-fail`
* `get-peg-in-sent-or-default`

### Relevant internal functions

* `index-tx`

## Contract calls (interactions)

* `executor-dao`: Calls are made to verify whether a certain contract-caller is designated as an extension.
* `meta-bridge-registry-v2-03`: This interaction is primarily utilized by the peg-out contract to register request operations. Additionally, this contract is responsible for managing approved addresses. It provides functionality through its `is-peg-in-address-approved` and `is-fulfill-address-approved` functions, which verify whether specific addresses are authorized within the system.
* `oracle-v2-01`: This contract is called to verify that a transaction was mined and to index Bitcoin transactions.
* Tokens (`token-trait`) During the steps of peg-out operations, this trait is utilized to invoke the relevant tokens and execute the necessary transfers involved in the transaction. It is a customized adaptation of Stacks' standard definition for Fungible Tokens (`sip-010`), with additional support for 8-digit fixed-point notation.
* `token-abtc`: This is the Bridged BTC token used throughout the peg-out contract to operate with native Stacks' SIP-010 token transactions involving the burn blockchain base-coin (BTC).

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


<!-- Documentation Contract Template v0.1.0 -->
