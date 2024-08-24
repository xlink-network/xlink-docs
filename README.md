---
description: XLink is not a typical cross-chain bridge
---

# What is XLink

## Key to providing the "native-like" Bitcoin DeFi experience

XLink is a key component of any projects building on Bitcoin that abstracts the difference between L1 and L2 from the user experience, i.e. providing the "native-like” Bitcoin DeFi experience on L1, whereby users can use native BTC or L1 assets issued on Bitcoin to interact with L2 smart contracts.

XLink is bi-directional or “two-way” bridge, meaning you can freely transfer assets between Bitcoin and its L2s and vice versa.

On Bitcoin, users interact with [Multisigs](./#multisigs) (operated by a decentralised network of validators and verifiers) to lock assets to be bridged ("source asset"), and on L2s to receive the L2 asset ("destination asset").

Additionally, users on Bitcoin may provide additional data (`OP_RETURN`) to trigger certain smart contract interaction on their behalf automatically by Bitcoin Bridge.

On L2s or non-Bitcoin chains, users interact with "[Endpoints](./#endpoints)" on the source blockchain to lock assets to be bridged ("source asset"), and on the destination blockchain to receive the bridged asset ("destination asset").

Asset transfers from users to Endpoints are monitored by a group of "[validators](./#validators)", who produce cryptographic proofs that say "X amount of source asset are sent by address A on the source blockchain for address B on the destination blockchain to receive Y amount of destination asset."

There is a minimum threshold of such proofs that must be provided and verified before the assets received into Endpoint can be sent to the relevant address.

Upon meeting such minimum threshold, a "[relayer](./#relayers)" calls into Endpoint with the proofs to trigger the transfer of the received destination assets to the relevant address.

## Multisigs

Multisigs are Bitcoin wallets that are operated by multiple signers. In contrast to a typicall wallet requiring just one party to sign a transaction, a multisig requires multiple parties or signers to sign a transaction.

## Endpoints

Endpoints are the smart contracts that handle the asset transfers. They are owned by multisig contracts (for example, [Gnosis Safe](https://safe.global/) on Ethereum and [Executor DAO](https://explorer.stacks.co/txid/0xf4bd95ea0486e6a50ae632c613f1d72b2a5bbbc4211b494cd0f1d3443658544d?chain=mainnet) on Stacks) operated by a decentralised network of validators and verifiers.

Users use Endpoints to trigger transfer of source assets. The destination assets are then sent by a relayer by producing cryptographic proofs.

That the assets are held by contracts owned by multisig contracts is important because this minimises the risk of a malicious actor stealing the private key and assets sent by users.

## Validators

Validators are responsible for producing cryptographic proofs, which must be verified by the Endpoints before transferring the destination assets to an address.

[Bitcoin Oracle](https://docs.alexgo.io/bitcoin-oracle/what-is-the-bitcoin-oracle) runs the validator network for XLink.

Validators are important because this set-up minimises the risk of a malicious actor taking over the system (for example, a relayer).

## Verifiers

Verifiers provide an additional level of protection by verifying those asset transfers requested by Relayers are good and not tempered. Verifiers thus complement the Endpoints verification by incorporating additional off-chain information which may not be available to the Endpoints.

[Bitcoin Orable](https://docs.alexgo.io/bitcoin-oracle/what-is-the-bitcoin-oracle) runs the verifier network for XLink.

## Relayers

Relayers are responsible for triggering the destination asset transfer upon gathering a minimum threshold of cryptographic proofs produced by the validators.

Relayers, however, cannot move the assets at will and their role is limited only to delivering messages from/to multisigs and Bridge endpoint.
