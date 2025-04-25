# Add a New Chain

<figure><img src="../.gitbook/assets/join the bridge.png" alt=""><figcaption></figcaption></figure>

Want to bring your chain into the Brotocol ecosystem?&#x20;

Whether you're an L1, L2, or appchain team, integrating with Brotocol opens your users to native Bitcoin liquidity, cross-chain swaps, and secure bridging—all while preserving Bitcoin as the settlement layer.

**Here’s how to get started:**

***

### 🧩 Overview

To support a new chain, Brotocol deploys a smart contract called an **Endpoint**, which acts as the entry and exit point for cross-chain asset transfers. These Endpoints are governed by multisigs (e.g. Gnosis Safe, ExecutorDAO) and coordinated through the **Bonbori** consensus layer.

Once deployed, your Endpoint becomes part of the BroBridge routing network—supporting swaps, payments, and bridging for supported assets.

***

### ✅ Integration Requirements

To be eligible for integration, your chain should support:

* ✅ Smart contract deployment (EVM-compatible or custom logic)
* ✅ Reliable block finality for message validation
* ✅ Token standards (ERC-20 or equivalent) for fungible assets
* ✅ A relayer-friendly RPC interface for transaction submissions

For Bitcoin L2s or non-EVM chains, custom integration paths are available. Reach out to the team for support.

***

### 🔧 Integration Steps

#### 1. Contact the Brotocol Team

Reach out via Discord or email to start the process. Provide basic info:

* Chain name & website
* RPC endpoints & explorers
* Token assets you want to support
* Dev contact for coordination

#### 2. Deploy Endpoint Contract

Brotocol will deploy an **Endpoint contract** to your chain. This contract:

* Receives incoming assets
* Validates cryptographic proofs from Bonbori validators
* Releases assets to users or contracts

Endpoints are owned by a multisig for security and flexibility.

#### 3. Configure the Endpoint

Once deployed, the following parameters are set:

* Approved token list
* Validator set & thresholds
* Fee structure
* Supported swap paths (optional)

#### 4. Enable Relayer Support

Brotocol relayers will monitor your chain’s Endpoint and submit proofs to/from other chains.

#### 5. (Optional) Deploy Synthetic Assets

If your chain doesn’t natively support a desired asset (e.g., BTC), a synthetic version (e.g., aBTC or aUSD) may be deployed and mapped to real assets via BroBridge.
