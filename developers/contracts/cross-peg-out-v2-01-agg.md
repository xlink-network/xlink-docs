# cross-peg-out-v2-01-agg.clar

- Location: `xlink/packages/contracts/bridge-stacks/contracts/btc-peg-out-v2-01-agg.clar`
- [Deployed contract]()

This technical document provides a detailed overview of the contract responsible for managing the peg-out aggregator process. The contract manages the transfer of tokens from Stacks to other blockchains by burning aBTC, employing non-ALEX liquidity aggregators, validating amounts and applying the necessary fees. The core functionality of the contract is implemented through the `transfer-to-swap` function. Lets examine the main components of the contract below.

## Storage

### `is-paused`
| Data     | Type   |
| -------- | ------ |
| Variable | `bool` |

A flag that indicates whether the peg-out process is active. If set to `true`, all peg-out operations are paused, preventing any new transactions. The contract is deployed in a paused state by default.

## Features

### Peg-out feature

#### `transfer-to-swap`
This function manages the peg-out process, enabling users to transfer bridged tokens from the Stacks network to other blockchains. It begins by validating the transfer through the `validate-transfer-to-swap` function, performing checks such as token and chain approval and amount thresholds.
Once validated, the function calculates the required fee and determines the net amount to be transferred. It accesses liquidity aggregators if necessary, and depending on the token's properties, the function either burns the net amount directly from the user's balance or transfers the entire amount (including the fee) to the `cross-bridge-registry-v2-01`.
At the end of the process, the function logs key destination details known as `settle-details`, including the `address`, which represents the recipient's address on the target blockchain.

### Governance Features

#### `is-dao-or-extension`
This standard protocol function checks whether a caller (`tx-sender`) is the DAO executor or an authorized extension, delegating the extensions check to the `executor-dao` contract.

##### Parameters
```lisp
(user principal)
```

#### `get-paused`
A read-only function that checks the operational status of the contract.

#### `set-paused`
A public function, governed through the `is-dao-or-extension`, that can change the contract's operational status.

##### Parameters
```lisp
(paused bool)
```

### Getters

#### `get-approved-chain-or-fail`
##### Parameters
```lisp
(dest-chain-id uint)
```
#### `get-token-reserve-or-default`
#### `get-min-fee-or-default`
#### `get-approved-pair-or-fail`
##### Parameters
```lisp
(pair { token: principal, chain-id: uint })
```

## Contract calls (interactions)
- `executor-dao`: Calls are made to verify whether a certain contract-caller is designated as an extension.
- `cross-bridge-registry-v2-01`: This contract is called to verify key components of the peg-out process. It validates approved tokens and chain pairings, manages the deduction and queries of token reserves, and records accrued fees. It also handles updates to transaction statuses.
- `token-in-trait`: In the cross-peg-out process, this trait is employed to manage token operations such as burning tokens when required or transferring them to the `cross-bridge-registry-v2-01` contract.

## Errors

| Error Name       | Value         |
| ---------------- | ------------- |
| `ERR-NOT-AUTHORIZED` | `(err u1000)` |
| `ERR-PAUSED`    | `(err u1015)` |
| `ERR-USER-NOT-WHITELISTED` | `(err u1016)` |
| `ERR-AMOUNT-LESS-THAN-MIN-FEE`    | `(err u1017)` |
| `ERR-INVALID-AMOUNT`   | `(err u1019)` |