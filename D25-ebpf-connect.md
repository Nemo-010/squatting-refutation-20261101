Part of the refutation sweep (#1).

**Claim under test.** `kprobe:tcp_connect` is the single best probe: populated `struct sock`, upstream's own `tcpconnect.bt`, catches every outbound TCP connection with address and port (`research/ebpf-runtime-detection-single-host.md` §3.1).

**The shipped one-liner is IPv4-only.** The inline probe in §3.1 filters `$sk->__sk_common.skc_family == 2`; upstream's `tcpconnect.bt` handles `AF_INET6` as well. A relay dialling its upstream over IPv6 — or a dual-stack host preferring AAAA — is dropped by the recommended one-liner while `tcp_connect` still fires. It also never resolves the v6 fields, so there is no v6 output to notice.

**P20 — it is a TCP probe.** QUIC, hysteria2, TUIC, WireGuard, `dnstt` and every UDP relay never call `tcp_connect`. The corpus has a separate UDP policy for Tetragon (§5.2) but the bpftrace section offers no UDP equivalent, and the "six probes cover everything" claim is a TCP claim.

**Start time.** A kprobe fires when the function is called. A tunnel that connected before `bpftrace` started is invisible until it reconnects; the corpus says this ("catches a new squat, not an old one") and then recommends the probe as the primary. This is a detector-start limitation, not a socket-reuse bypass.

**P06 — connect elsewhere.** The fd is connected by a helper and passed in. The probe fires, with the helper's `comm` and `pid`, which have already exited.

**Requires privilege and a clean kernel.** No `CAP_BPF`/`CAP_SYS_ADMIN` means no probe. `kernel.unprivileged_bpf_disabled` is why those capabilities are required (default 2 refuses unprivileged loads outright); it is not a second, independent obstacle once the caller has the cap. Lockdown `confidentiality` blocks it too.

**P04.** Root can stop the tracer, unload the program, or attach its own probes. The probe is not tamper-evident.

**Verdict: medium-high.** It is genuinely the best *TCP* on-host view and the corpus is right about that. The gaps — UDP, pre-existing flows, helper processes — are the three things a relay written by someone who read this document would choose.
