# Integrations

Integration requires a deployment of an endpoint smart contract on the target chain.

For the list of currently supported chains, please visit Brotocol[ Contract Deployment](https://app.gitbook.com/s/xagAneFBZMxG6fw5k0WK/).

Below is a typical integration process.

## Endpoint deployment

Endpoints are the smart contracts that handle the asset transfers. They are owned by multisig contracts (for example, [Gnosis Safe](https://safe.global/) on Ethereum and [Executor DAO](https://explorer.stacks.co/txid/0xf4bd95ea0486e6a50ae632c613f1d72b2a5bbbc4211b494cd0f1d3443658544d?chain=mainnet) on Stacks) operated by a decentralised network of validators and verifiers.

Users use Endpoints to trigger transfer of source assets. The destination assets are then sent by a relayer by producing cryptographic proofs.

## Configuration of endpoint

Once the endpoint is deployed, it can be configured to meet the needs of the source chain.

The configuration parameters include, among others,:

* Approved list of tokens to be supported on the Endpoint,
* Approved list of validators whose cryptographic proofs of token transfer event are accepted,
* Approved list of relayers who can submit the cryptographic proofs from the validators,
* Validator threshold, and
* Fee schedule

## Integration with Bitcoin Oracle

Brotocol scales by partnering with [Bitcoin Oracle](https://docs.alexgo.io/bitcoin-oracle/what-is-the-bitcoin-oracle) which runs the validator and verifier networks of Brotocol.

Bitcoin Oracle produces the consensus data based on the computation from the off-chain engines and provides a consensus model framework that allows the end consumer to customise their consensus model by optimising across the trust and the security budget.

The validators of Bitcoin Oracle observe every Endpoint on the Brotocol and produce a set of cryptographic proofs for the relevant destination chain to process.

The relayers then aggregate those cryptographic proofs and submit to the relevant destination Endpoints upon meeting the validator threshold.

The submissions by the relayers are verified by the verifiers, which act as the additional protection to the verification by the Endpoint.

## Synthetic asset deployment (optional)

Where required, a synthetic asset may be deployed on the destination chains to allow the bridging of the asset from its source chain to the destimation chains.
