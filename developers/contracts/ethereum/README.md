# Technical Documentation for Smart Contracts on Ethereum

This document outlines the main functionalities of the XLink Bridge as deployed on the Ethereum blockchain. The core aspects of the bridging operation are implemented in the `BridgeEndpoint` contract, which acts as the main entry and exit point for cross-chain transactions, ensuring that assets are securely locked, transferred, and released. The `BridgeEndpointWithSwap` contract extends `BridgeEndpoint`, incorporating swaps during bridging. It ensures that a token swap and bridge operation occur in a single transaction.

## Smart Contracts on Ethereum

{% content-ref url="BridgeEndpoint.md" %} [BridgeEndpoint.md](BridgeEndpoint.md) {% endcontent-ref %}

## Main Features

The XLink ecosystem offers two main features in its Ethereum implementation. These are:
- The **bridging of EVM-like blockchain's assets**: these contracts interact with XLink smart contracts on Stacks to allow the transfer of assets back and forth between EVM-compatible blockchains and Stacks. 
- The **swapping of tokens in the bridging process** via external liquidity aggregators.

