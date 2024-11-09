TLDR:
Bitcoin custodian solvency can't be fully audited because custodians control both assets and securities issuance. In contrast, non-custodial "layers" follow consensus rules and don’t require auditing.

True solvency proof needs simultaneous verification of assets and issued securities, which is impractical, especially for national reserves involving Bitcoin and fiat. Even if deviations are detected, history shows the problem isn't finding them—it’s stopping them.

--------

Solvency of a Bitcoin custodian cannot be audited. A custodian is a [person](Glossary#person) with discretion both in the release of an asset and issuance of securities against it. If both release of the asset and the issuance of securities against it are controlled by [consensus rules](Glossary#consensus-rules), then the relationship is not actually custodial. This is the distinction between a [reserve](Reservation-Principle) and a [layer](Glossary#layering). A layer is protocol-enforced (non-custodial) and therefore has nothing to audit.

**A solvency audit requires simultaneous (atomic) proof of both the full amount of the asset held by a custodian and the securities issued against it.** In the case of a national Bitcoin reserve this would require complete proof of all fiat (e.g. the security) issued against the reserve, as well as the Bitcoin held in reserve. Even in the case where the security is issued on a distinct public [chain](Glossary#chain) the atomicity requirement is not satisfied.

In some cases it may be considered sufficient to waive the atomicity requirement, accepting incorrectness under the assumption that material deviation would eventually be discovered. However in the case of [state banking](Reserve-Currency-Fallacy) it is insufficient to detect the deviation. Historically it has not been difficult to detect such deviations. The difficulty arises in stopping them.
