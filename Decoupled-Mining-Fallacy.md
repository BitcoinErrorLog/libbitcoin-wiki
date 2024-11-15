TLDR:

The theory that decoupling reward from transaction selection in pooled mining increases security is flawed. It wrongly assumes transaction control shifts to smaller miners, enhancing censorship resistance. In reality, pool operators retain control, making decoupled pools as vulnerable to censorship and co-option as traditional coupled pools.

Even with transparency, a decoupled pool doesn't prevent state co-option or censorship. The core issue remains: miners give up control to a central pool operator, much like in the Relay Fallacy, meaning any supposed security gains are illusory.

--------

There is a theory that [security](Qualitative-Security-Model) is increased by decoupling [reward](Glossary#reward) from [transaction](Glossary#transaction) selection in [pooled](Glossary#pooling) [mining](Glossary#mine). The theory holds that by sharing only the reward, control over transaction selection shifts to [miners](Glossary#miner) with less [hash power](Glossary#hash-power). This implies a reduction in the [variance discount](Variance-Discount-Flaw) and therefore an increase the [competitiveness](Censorship-Resistance-Property) of smaller mines. Because smaller mines can presumably operate more covertly than larger, this in turn implies that [censorship](Glossary#censorship) [resistance](Axiom-of-Resistance) is increased.

**The theory fails to recognize that control over transaction selection remains with the pool operator**, and is therefore invalid. The sole benefit is [variance](Glossary#variance) reduction, but this is only realized by the receipt of payment. As payment is discretionary any condition may be attached. Such conditions may include censorship and [identity](Glossary#identity). Member recourse is to leave the pool for another, just as with a coupled pool. As such [decoupled](Glossary#decouple) pools and coupled pools are equally subject to [co-option](Glossary#co-option).

There is a related theory that transparency of a decoupled pool is greater than that of a coupled pool, facilitating flight of members to non-censoring pools, therefore limiting the dominance of censoring pools. Generously accepting the assumptions of greater transparency and independent miners operating against financial self-interest, we are still left with the fact of co-option. The [state](Glossary#state) can still reserve for itself the ability to operate with the financial advantages of [pooling](Pooling-Pressure-Risk) and the theory is therefore invalid.

This fallacy is similar to the [Relay Fallacy](Relay-Fallacy) in that all financial advantage depends on otherwise independent miners granting control over that advantage to a single [person](Glossary#person).
