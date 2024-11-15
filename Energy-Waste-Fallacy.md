TLDR:

The theory claims Proof-of-Work (PoW) wastes energy, suggesting either excessive security or that another proof could achieve similar security at lower cost. This theory ignores market-driven factors: hash power follows rewards, and security demand defines transaction fees. Security is subjective and linked to confirmation depth and utility thresholds.

Replacing PoW's energy cost with other resource costs doesn’t solve energy concerns, as all costs eventually resolve back to energy. The market implies Bitcoin's current form is preferred due to overall cost efficiency, meaning alternatives are likely less energy efficient. Ultimately, energy use in securing Bitcoin is not inherently wasteful but rather reflects optimal resource allocation for security.

--------

There is a theory that [proof-of-work](Glossary#proof-of-work) wastes energy. This implies that the level of security provided is greater than necessary or the same level of security can be provided by another externalized [proof](Glossary#proof) at a lower energy cost. An *internalized* proof, specifically [proof-of-stake](https://wikipedia.org/wiki/Proof-of-stake) (PoS), is a different security model which is not [cryptodynamically secure](Proof-Of-Stake Fallacy), and is not considered here.

Total [hash power](Glossary#hash-power) is a function of [reward](Glossary#reward), which is a function of [fees](Glossary#fee), which are determined by the [confirmation](Glossary#confirmation) [market](Glossary#market). If a [person](Glossary#person) considers current hash power insufficient to secure [trade](Glossary#trade) at a given [value](Glossary#value) against [double spend](Glossary#double-spend) then the [depth](Glossary#depth) requirement increases. Additionally, as shown in [Utility Threshold Property](Utility-Threshold-Property), [transactions](Glossary#transaction) with insufficient value for even single confirmation security are [priced](Glossary#price) out of the [chain](Glossary#chain).

These upper and lower security bounds depend on confirmation cost and are therefore independent of the proof technique. **There is no *necessary* level of security, just a subjective confirmation depth and minimum utility.**

Confirmation security increases with the cost of generating each [block](Glossary#block). The double spend of a transaction requires that its [branch](Glossary#branch) be superseded by another with a probabilistically greater cost. So energy cost can be reduced only by expending the same average cost for a given confirmation time, but with a lower energy component.

[Work](Glossary#work) incurs cost in several forms, including labor, hardware, services, land, etc. Any other externalized proof consumes these same resources, though potentially in different proportion. The question of energy cost reduction is therefore reduced to whether an energy component of the cost of a proof can be replaced by an other resource component with the same cost. However the cost of the substitute resource includes all of its production costs, which must resolve to energy. The theory is therefore invalid.

Additionally, securing any [coin](Glossary#coin) incurs a cost to [merchants](Glossary#merchant). As such the fact of its use by them implies that it is preferred over alternatives. This implies the alternatives are ultimately more costly. As all costs are fundamentally resolved in energy consumption, it follows that the [money](Money-Taxonomy) in use is the most energy efficient.
