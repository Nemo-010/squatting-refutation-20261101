Part of the refutation sweep (#1).

**Claim under test.** `ss -H -0 -p` lists `AF_PACKET` sockets, so a squatter sniffing traffic — or a sniffer nobody installed — is visible (`checks/triage.sh`).

**The premise is wrong for this threat.** A proxy relay does not need to capture packets. Proxyjacking, a CDN tunnel, a seedbox and a restream are all ordinary socket users; the packet-socket sweep finds none of them. What it finds is an actual sniffer, which is a different crime with a different detection.

**`-0` is `-f link` only.** A userland TCP stack can use `AF_INET`/`SOCK_RAW` with `IP_HDRINCL` — no `AF_PACKET`, not in this view — and `AF_XDP` sockets are a third family again. An eBPF XDP program captures without any socket at all.

**Ephemeral.** Open the packet socket, capture the handshake you need, close it. The sweep is a snapshot; a one-second capture between two triage runs leaves no row.

**P04.** An LKM removes the socket from `ss`'s view, which is a smaller job than hiding a process.

**Verdict: low.** Correct command, wrong target. It belongs in a "is someone sniffing this host" checklist, not in a squatting checklist, and listing it under triage suggests a coverage it does not provide. The one thing it does add — evidence that the box's own capture tooling could be subverted — is already argued in D09.
