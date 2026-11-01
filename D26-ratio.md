Part of the refutation sweep (#1).

**Claim under test.** Aggregate `connect`s per `comm` over 60 s; a `comm` that connects but never accepts is the outbound-only shape (`research/ebpf-runtime-detection-single-host.md` §3.7).

**P18 — add an accept.** Open a loopback listener and `accept()` on it periodically (a health endpoint, or accept-and-close). The ratio now has an accept for every window, and the process looks like a server with client behaviour — which is what a normal application looks like.

**P13 — merge the counters.** `comm` is 16 bytes and free. Name the payload `nginx`, `node` or `postgres` and its connects aggregate with the real process's, which does accept. The detector is per-`comm`, not per-`exe`, so the merge is exact.

**P14 — a SOCKS relay is a server.** The most common proxyjacking payload binds loopback and accepts *inside* the tunnel; it also accepts from the local client. It has accepts, and it has connects. The shape is a normal proxy, which is what it is.

**UDP usually has no accept.** Connected UDP sockets exist, and the corpus's §5.2 policy keys on `security_socket_connect` with `IPPROTO_UDP`, so a UDP flow can produce a connect event. What it never produces is `inet_csk_accept` or `tcp_connect`, so this ratio — built from `tcp_connect` against `inet_csk_accept` — never sees QUIC, hysteria2 or `dnstt`.

**P17.** Reconnecting rarely does not evade a rule that fires on any non-zero count in a window: the window with a connect still fires. What does evade it is moving the connect out of this host's `comm` (P06) or out of this netns (P10); the rate framing only holds against an implementation that thresholds on connects per interval.

**Verdict: medium.** The ratio is a heuristic on a name, and both inputs — the name and the accept — are cheap to control. The corpus's own caveat ("a DNS client, an HTTP client with a long-lived outbound session, and anything behind a reverse proxy all look the same") is the honest statement; the recommendation to "confirm against the binary path" moves the whole weight onto D01, which has its own bypasses.
