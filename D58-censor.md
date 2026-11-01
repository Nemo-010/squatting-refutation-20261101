Part of the refutation sweep (#1).

**Claim under test.** There is a second detector in the path — the Chinese filtering system — whose first-payload classifier (protocol fingerprint, bit fraction, printable-ASCII statistics, ~50-byte probes) explains why the protocols look the way they do, with ~0.6 % collateral blocking and ~26 % probabilistic blocking of popular datacenter ranges (`REPORT.md` §3.5).

**This is the adversary's design driver, and every defence against it is a defence against the operator.** REALITY exists to be byte-identical to a real site; uTLS exists to stop Go's ClientHello sticking out; XHTTP exists to look like a browser's HTTP. Each of those was built to defeat a classifier that examines the same features an operator's JA3/JA4 rule examines. Strengthening the tunnel against the censor strengthens it against D31–D38.

**The classifier's false positives are the operator's too.** The paper's ~0.6 % collateral and ~26 % datacenter blocking describe a detector that blocks *by range and by statistics*. Any rule an operator writes on the same features inherits the same error rates on the same traffic.

**It is not available to the operator.** The classifier is a network-side system the operator does not run and cannot query. As a detection it contributes one thing: evidence that flow-shape detection has already been studied by people with more data, and that the state of the art moved to *making the flow indistinguishable* rather than *detecting it*.

**Verdict: not a detection the operator can use — and the corpus is honest about that.** Its real value in the document is explanatory: it is the reason a naive JA3 denylist loses, and the reason "indistinguishable from a browser" is a design goal rather than an accident. That argument should be applied to the operator's own §4, where several rules are the same kind of statistic at a smaller scale.
