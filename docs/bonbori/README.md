# What is Bonbori?

<figure><img src="../.gitbook/assets/bonbori.png" alt=""><figcaption></figcaption></figure>

Bonbori is a cross-chain messaging and consensus layer designed specifically for Bitcoin's off-chain computation ecosystem. The system:

* Creates a standardized framework for validating on-chain events across multiple protocols, such as BRC20, Runes, and Staking
* Enables secure message exchange between different off-chain computation engines
* Provides a decentralized validation mechanism with cryptographic attestations
* Maintains a complete audit trail for all validations

Bonbori serves as the critical foundation for each of Brotocol's core products:

1. **Multi-Chain Connector**: When Brotocol facilitates movement of stablecoins and other assets onto Bitcoin, Bonbori validates these cross-chain transfers through its threshold-based consensus system, ensuring all assets maintain integrity and security.
2. **Account Abstraction DEX**: Brotocol's DEX relies on Bonbori to securely tap into liquidity pools across chains like Base and Arbitrum. When a Bitcoin user initiates a trade from their Bitcoin wallet, Bonbori validates the transaction parameters before execution, ensuring optimal rates without compromising security.
3. **Payment Protocol**: As Brotocol enables non-Bitcoin users to purchase Bitcoin assets using currencies like USDC or ETH, Bonbori provides the attestation layer that verifies these cross-chain payment flows, allowing merchants to confidently receive BTC while buyers use their preferred currency.

### What does Bonbori do for Brotocol?

Here’s how Bonbori supports Brotocol’s core products:

**🔁 For BroBridge:**

* Validates asset transfers across Bitcoin, Stacks, EVM chains, and beyond.
* Produces cryptographic attestations confirming peg-in and peg-out events.
* Ensures that bridging is secure, atomic, and cryptographically verifiable.
* Supports threshold-based consensus (customizable per integration) to validate user intents and unlock assets on destination chains.

**💱 For BroSwap:**

* Verifies cross-chain trade intents expressed by Bitcoin users.
* Ensures that DEX aggregations and swaps are executed only after proper consensus is reached on transaction validity.
* Confirms final trade state and delivers assets back to Bitcoin wallets after cross-chain execution.

**💸 For BroPay:**

* Validates cross-chain payment flows when non-Bitcoin users pay with tokens like USDC or ETH.
* Confirms that a user's payment has been correctly routed and converted before releasing pure BTC to the merchant.
* Ensures end-to-end accountability and enables seamless refunds in case of failure.

**🛡️ As Infrastructure:**

* Provides **standardized validation and messaging** across all Brotocol integrations (Bitcoin L1, Bitcoin L2s, EVMs).
* Allows validators and verifiers to **observe and attest** to on-chain events.
* Lets **relayers submit proofs** and verifiers double-check them before releasing funds or confirming transactions.
* Supports **OP\_RETURN-based automations** triggered directly from Bitcoin L1.

### What Bonbori Means <a href="#what-bonbori-means" id="what-bonbori-means"></a>

Bonbori (雪洞) refers to traditional Japanese paper lanterns that illuminate pathways during festivals and ceremonies.&#x20;

Much like these lanterns guide people through darkness, Bonbori guides blockchain data across protocols with clarity and security—illuminating the path for Bitcoin's cross-chain communications.

