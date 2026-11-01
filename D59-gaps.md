Part of the refutation sweep (#1). This one is about what is *missing*, because a corpus that lists fifteen niches still leaves the cheapest channels out.

**Channels with no detection at all in the corpus:**

- **HTTP CONNECT and `ssh -D`.** The two most common proxy methods after SOCKS5, and neither has a byte constant or a Zeek note here.
- **DoH/DoT tunnels.** A tunnel inside DNS-over-HTTPS is a TLS session to a resolver on 443; it defeats D36 entirely and is not mentioned.
- **ICMP tunnels.** `ptunnel`/`icmptunnel` need no TCP or UDP port; nothing in the corpus looks at ICMP payload volume.
- **`sendmsg` with ancillary data / `MSG_ZEROCOPY`.** The audit rules match `connect`/`bind`; data-plane volume through `sendmsg` on a connected socket has no rule.
- **`AF_VSOCK`.** A relay to a co-resident VM over vsock never touches an IP address; no check here covers it.
- **User namespaces.** `unshare -Ur` gives a tenant a namespace where they hold `CAP_NET_ADMIN`; the corpus discusses containers but not the rootless-namespace path that produces them without a runtime.
- **io_uring *creation*.** Every io_uring bypass in this sweep is detectable at `io_uring_setup`/`io_uring_enter`, which the corpus monitors for `openat` only (D23).
- **Provider flow logs.** VPC flow logs, AWS Guard Duty, GCP VPC Flow Logs — the one place an operator can see a container's traffic without the guest's cooperation. Not mentioned.
- **Egress baselines per workload class.** "Traffic that does not match the unit's declared purpose" is asserted as the correlation that works, and no method for establishing the baseline is given.

**Channels covered thinly:** DNS (acknowledged), Windows-side (acknowledged), IPv6 in every blocklist and hex-port check, and TLS-in-TLS detection, which the corpus does not attempt despite it being the only positive signal that survives uTLS.

**Not a bypass, a structural point:** every check here reads the host the squat is on. The one view the squat cannot edit is a second machine's flow records and the provider's billing. The corpus says this and then spends most of its length on host-side checks. The imbalance is the finding.

**Closing note.** Nine of the channels above have no detection in the corpus at all, and the cheapest of them are also the quietest: DoH puts the tunnel inside the one protocol an egress allow-list has to permit, a user namespace produces a container without a runtime to inventory, and a provider's flow log is the one record an operator can read without the guest's cooperation. A second edition should start from these, not from a longer port table.
