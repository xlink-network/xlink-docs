# Bonbori Consensus Model

<figure><img src="../.gitbook/assets/consesus.png" alt=""><figcaption></figcaption></figure>

Bonbori uses '**Flexible Threshold-based Consensus Model**' which empowers Bitcoin builders with a customizable consensus framework that balances security needs with practical considerations:

### "M-of-N" Threshold Configuration <a href="#m-of-n-threshold-configuration" id="m-of-n-threshold-configuration"></a>

Bonbori's threshold configuration allows end consumers to tailor consensus requirements to their specific needs. Developers can define both required (trusted) validators and optional validators within their consensus model, then set specific agreement thresholds (such as requiring 51% of all validators to concur).&#x20;

This customizable approach enables applications to fine-tune their security parameters based on their unique risk profiles, transaction values, and operational requirements.

### Consensus Flexibility <a href="#consensus-flexibility" id="consensus-flexibility"></a>

Bonbori offers multiple consensus approaches to accommodate diverse security philosophies. Applications can implement a federated option that relies solely on required validators, prioritizing trust relationships to minimize security costs.&#x20;

Alternatively, they can choose a fully trustless option that utilizes optional validators with staking/slashing mechanisms for a permissionless system.&#x20;

Most applications benefit from hybrid approaches that combine trusted entities with broader validation, striking an optimal balance between security guarantees and operational efficiency.

### Threshold Sampling for Efficiency <a href="#threshold-sampling-for-efficiency" id="threshold-sampling-for-efficiency"></a>

Bonbori achieves consensus efficiently through its innovative threshold sampling approach. Rather than requiring validators to download and process all consensus data, the system conducts multiple rounds of random sampling across network nodes.&#x20;

With each successful round, consensus confidence grows incrementally until reaching a predetermined threshold. This progressive sampling dramatically reduces bandwidth and computational requirements while maintaining security integrity.&#x20;

The final validation always incorporates verification by trusted nodes, ensuring that efficiency never comes at the expense of reliability.

