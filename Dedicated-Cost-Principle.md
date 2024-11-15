TLDR:

Unnecessary miner costs, like misconfigured machines, are waste—they don’t improve double-spend or censorship resistance. Only essential costs that efficiently produce hash power contribute to security.

The idea of using Proof-of-Work to generate valuable byproducts (e.g., prime numbers) is flawed. Competition still drives costs to match rewards, making any byproduct irrelevant.

Merged mining also fails to improve new coin security: hash power earned from another chain doesn’t add to the new coin's security and can be censored at no additional cost.

--------

Unnecessary costs incurred by [miners](Glossary#miner) contribute nothing to either double-spend resistance or [censorship resistance](Censorship-Resistance-Property). Such costs constitute true waste, representing nothing more than a given miner's inefficiency. For example, it does not contribute to security if a miner with misconfigured [machines](Glossary#machine) expends a great deal of energy while being unable to win a [reward](Glossary#reward) due to the misconfiguration. Any cost that is not strictly required for the optimal generation of [hash power](Glossary#hash-power) is not a necessary cost. A misconfiguration of one miner does not represent cost to another.

There is a theory that [proof-of-work](Glossary#proof-of-work) (PoW) can be made more [energy efficient](https://github.com/libbitcoin/libbitcoin-system/wiki/Efficiency-Paradox) by introducing non-dedicated costs to the [mining](Glossary#mine) function. One such example is the discovery of [prime numbers](http://primecoin.io). The reason to incorporate such costs is that the resulting discoveries have presumed marketable value. If not there would objectively be no value in the incorporation.

By analogy, brewers can sell their grain byproducts to farmers. This improves their efficiency by eliminating an unnecessary cost. So to the extent that the resulting byproduct is valuable, it's production does not incur a net cost. Yet necessary net cost must rise to the level of reward, as a consequence of competition. Therefore the same result would be achieved by basic PoW consuming the full reward value and independent energy-consuming operations generating the marketable products. **Any cost dedicated to the production of independently-marketable value can be offset by selling that byproduct.** As such the theory is invalid.

[Merged mining](https://eprint.iacr.org/2017/791.pdf) is typically implemented to resolve the problem of bootstrapping a new coin past the vulnerable stage of low [hash rate](Glossary#hash-rate). This design fails to recognize that hash rate not dedicated to the new coin does not contribute to its security. As the full cost of the [hash rate](Glossary#hash-rate) can be recouped by selling it on one [chain](Glossary#chain), there is no cost to [censor](Glossary#censorship) the other merge-mined chain(s).
