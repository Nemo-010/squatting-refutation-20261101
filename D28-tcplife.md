Part of the refutation sweep (#1).

**Claim under test.** bcc's `tcplife` is the only free on-host way to measure bandwidth per connection, and `bindsnoop` is the transparent-proxy fingerprint at bind time (`research/ebpf-runtime-detection-single-host.md` §4).

**tcplife traces state changes, not packets.** It hooks `sock:inet_sock_set_state` (kernel ≥ 4.16). A connection that was already `ESTABLISHED` when the tool started has no state change to observe until it closes — which may be never. The corpus's own summary says "per-connection byte counters"; the implementation is per-transition.

**It is TCP.** A QUIC tunnel is invisible; `tcplife` has no UDP twin in the corpus.

**Short flows do not defeat it.** `tcplife` emits a record on each session close, reading the socket's `tcp_info`; reconnecting every 30 s produces *more* records, and the summed bytes still reconcile. Short flows defeat a *rate* or *long-lived-flow* reading of the output, not the accounting. The real gap is the one above: a flow that never transitions is never printed.

**bindsnoop sees `bind()`, not `connect()`.** An outbound-only relay never binds. The fingerprint (`IP_TRANSPARENT`, `IP_FREEBIND`) is for a *transparent* proxy, a design most relay payloads do not use. On the population the document describes, `bindsnoop` prints nothing.

**P04.** Both are bcc tools with the same privilege and tamper properties as D25.

**Verdict: medium for `tcplife` (with the "already established" caveat stated), low for `bindsnoop`.** The pair is presented as a measurement layer; one of them measures a transition and the other measures a design choice that the corpus's own examples do not make.
