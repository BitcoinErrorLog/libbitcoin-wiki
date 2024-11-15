TLDR:

The theory that immutable claims can secure property ownership, including against custodial risk, misunderstands the nature of claims. Control always rests with the custodian, making them a trusted third party. Immutable claims do not protect against custodian failure or state intervention, as seen with historical examples of state seizure.

While immutable storage might prevent claim data loss, owners still need to prove their ownership to the custodian, so the security risk just shifts forms. Bitcoin avoids these issues by being non-custodial—its value is held by the merchants and users themselves. The "blockchain fallacy" wrongly attributes Bitcoin’s security to its data structure rather than its distributed, non-custodial nature.

--------

There is a theory that property [ownership](Glossary#owner) can be secured by immutable [claim-keeping](Glossary#claimant), both against claim loss and [Custodial Risk](Custodial-Risk-Principle).

Given that a claim is not itself the property, control of the property rests with the [custodian](Glossary#custodian) against whom the claim is made. A custodian has the ability to surrender or retain the property and is therefore a [trusted third party](https://en.wikipedia.org/wiki/Trusted_third_party). Abrogation of a claim by its custodian is always mitigated by custodian signature, cryptographic or otherwise, with enforcement of the claim left to its holder.

The theory asserts that immutable claim-keeping provides security against loss of the claim by its owner, as nobody else would have an interest in the loss. However, in order to redeem the claim its owner must produce proof of ownership to the custodian. This requires that the owner not lose the secret that proves this ownership.  As such the security of the claim against loss is not mitigated at all, it merely changes form. The theory is therefore invalid on the basis of loss prevention.

Storing a strong reference to the claim can reduce the size, and therefore cost, of its immutable storage. The claim may be in the form of a [human](Glossary#person) or [machine](Glossary#machine) [contract](Glossary#contract), and referenced as a [one way hash](https://en.wikipedia.org/wiki/Cryptographic_hash_function). In either case the [validation](Glossary#validation) and execution of the contract is required for property transfer by the custodian. Therefore a referenced contract claim compounds loss risk with additional data, the contract.

As shown in [Risk Sharing Principle](Risk-Sharing-Principle), people are always the basis of security. People may act collectively to protect the immutability of a money, and therefore any claim data associated with control of the money. 

However, a custodian is a trusted third party. Immutable claims do not in any way mitigate direct attacks against, or by, a custodian. Where the custodian is the [state](Glossary#state) or is subject to its control, the claim offers [no security](https://en.m.wikipedia.org/wiki/Executive_Order_6102) against the substitution of state authority in place of proven ownership of any claim. The theory is therefore also invalid on the basis of custodial failure.

Bitcoin as a [money](Money-Taxonomy) is non-custodial. Its [units](Glossary#unit) do not represent an asset held by a trusted third party. The money is traded directly between customer and [merchant](Glossary#merchant). In this sense *all merchants* are custodians of Bitcoin's [value](Glossary#value). **The blockchain fallacy arises from a misconception of the Bitcoin security model, attributing security to its technology as opposed to its distribution of merchants.** The term "blockchain technology" reinforces this error, implying that it is primarily the structure of Bitcoin's data that secures it.
