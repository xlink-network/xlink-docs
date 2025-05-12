# Technical Documentation for Smart Contracts on Ethereum

This document outlines the main functionalities of the XLink Bridge as deployed on EVM-compatible blockchains. The core aspects of the bridging operation are implemented in the `BridgeEndpoint` contract, which acts as the main entry and exit point for cross-chain operations, ensuring that assets are securely locked, minted/burned, transferred, and released. The `BridgeEndpointWithSwap` contract extends this functionality by incorporating swaps, allowing bridging and swapping to occur in within a single transaction. 

## Contracts

{% content-ref url="BridgeEndpoint.md" %} [BridgeEndpoint.md](BridgeEndpoint.md) {% endcontent-ref %}

## Main Features

The XLink ecosystem offers two main features in its EVM implementation:
- The **bridging of ERC-20 assets**: these contracts interact with XLink's off-chain protocol components to allow the transfer of assets back and forth between EVM-compatible blockchains and Stacks.
- The **swapping of tokens in the bridging process** via external liquidity aggregators.

### EVM Chain Bridge

![This is a simplified representation on the EVM Chain Bridge's main goal.](../../../.gitbook/assets/glue-docs/bridge-endpoint.png)

\*To see more information on these contracts see the [auxiliary contracts section](#auxiliary-contracts).</small>

#### Bridge Endpoint

- Contract names: `BridgeEndpoint`, `BridgeEndpointWithSwap`.
- [Complete technical documentation](BridgeEndpoint.md)

This endpoint's main responsibility is serving as the entry and exit point for assets moving along the Cross Chain Bridge. Sometimes, it also involves swapping to other tokens as part of the peg-in process.

### Auxiliary Contracts

These contracts do not include the implementation of any core functionality but they serve as a support for other contracts to facilitate calculations and common storage management.

- **Bridge Registry**: this contract keeps a record of approved tokens, validators, relayers and fees. It also keeps a record of the generated orders and their statuses. Its latest version can be found at `BridgeRegistry.sol`.