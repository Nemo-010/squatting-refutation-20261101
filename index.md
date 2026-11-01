# The refutation sweep

Fifty-nine issues against the detections in [squatting-20261031](https://github.com/Nemo-010/squatting-20261031),
one per detection family, each naming the cheapest way past it. Method, verdict legend and the checks
that were re-verified first-hand are in [README.md](README.md).

This is the adversarial index for the corpus. Every detection family in `REPORT.md`, `research/` and `checks/` gets one issue below, and every issue records the ways a squatter gets past it.

Two ground rules, because a refutation document is worthless if it flatters itself:

- **A bypass is only real if it is cheaper than the detection.** Each one names its prerequisite. "Root can do anything" is not an interesting finding and is not counted as one.
- **Where a squat is genuinely indistinguishable from legitimate use, the issue says so** instead of inventing a residual signal. Those carry the `indistinguishable` label.

The issues are descriptive. They are written for the person running the detector, so the detector can be fixed; they contain no working payloads.

## Bypass primitives

Cross-cutting mechanisms, referenced by number from the detection issues. The point of listing them once is that most detections fail to the same dozen moves.

**P01 — Execute from a path nobody enumerates.** The proc sweep matches `/tmp`, `/var/tmp`, `/dev/shm`, `/run`, `~/.cache`, `(deleted)` and `/memfd:`. It does not match `$HOME/.local/bin`, `$HOME/.local/lib/pythonX/site-packages`, `/opt`, `/srv`, `/usr/local/lib` or `/var/lib/<name>` — all writable by the tenant. `pip install --user` and `npm --prefix` produce exactly that layout, so the binary lives in a "software" directory and the sweep reports nothing.

**P02 — Do not create an executable file at all.** `python3 -c`, `node -e`, `perl -e`, `java -jar`, or a shared object `dlopen`ed into an existing process (a Postgres extension, an nginx module, a Jupyter kernel) leaves no new image on disk and no `execve` of a payload. Exec-based detection — auditd, execsnoop, Tracee's fileless and dropped-executable signatures — has nothing to fire on.

**P03 — Hook the reader.** `LD_PRELOAD` (per user, via `.bashrc`) or `/etc/ld.so.preload` (system-wide) intercepts `fopen`/`open`/`readdir`/`lstat` and rewrites what the checking tool sees. `/proc/net/tcp` is trivially filtered; netlink can be filtered at `sendmsg`/`recvmsg`; `ss -p` resolves owners through `opendir("/proc")`, so process attribution is hookable even when the socket list is not.

**P04 — Hide in the kernel.** An LKM patching `getdents`/`kill`, or an eBPF program attached ahead of the operator's, defeats every userland view at once. The only survivors are counters the operator owns outside the host (provider billing, a second machine's flow records).

**P05 — io_uring.** `IORING_OP_CONNECT`, `SOCKET`, `BIND` and `SENDMSG` run from io-wq/kernel context with no syscall entry. The `exit` filter therefore never sees them; only rules on the separate `io_uring` filter match, and those must name each opcode individually. Tetragon/kprobe probes at `tcp_connect` still fire, which is why P05 pairs with P10.

**P06 — Pass the file descriptor.** `SCM_RIGHTS` over a Unix socket hands a live connected socket to another process; the creator closes its copy and exits. The socket then appears only under a process that looks legitimate, audit's PID correlation no longer ties creation to use, and `ss -p` shows the holder rather than the author.

**P07 — Reuse a socket you did not create.** systemd socket activation, a listener opened at boot, or `accept()` on an existing listener produces no `socket()` and no `connect()` event. Count-per-connect and connect-only-ratio rules see nothing new.

**P08 — `clone3` for namespaces.** `-S clone -F a0&0x40000000` matches `clone`, not `clone3`, and `clone3`'s flags live in a struct behind a pointer, so the `a0` mask cannot be applied at all. A netns created through `clone3(CLONE_NEWNET)` leaves no audit event under that ruleset.

**P09 — Bind-mount over a watched directory.** Audit watches are installed on inodes. `mount --bind` a different tree over `/etc/systemd/system` (root, `CAP_SYS_ADMIN`) and writes land in an unwatched directory at a watched path.

**P10 — Run it in another namespace or cgroup.** A container pod, a rootless podman container, a nested VM. Host `ss` cannot see another netns; `meta skuid` needs the socket's netns to match; a per-unit `IPEgressBytes` does not include a sibling scope.

**P11 — Use a different interface than the one being counted.** macvlan/ipvlan and SR-IOV VFs hand the tenant a device that transmits without passing the host interface's counters; a VM with a passthrough device does the same one level up.

**P12 — Spawn outside the metered unit.** `systemd-run --scope`, `systemd --user`, or a rootless container creates a new scope under `user.slice` rather than under the notebook's service. Per-unit accounting is blind to it; only a whole-host sum sees the bytes.

**P13 — Masquerade.** `prctl(PR_SET_NAME)` sets `comm` (16 bytes), `exec -a` sets `argv[0]`, and `cmdline` is explicitly "what the process wants you to see". Copy the binary to a plausible name and it matches none of the name rules.

**P14 — Userspace proxy, no TUN.** `tailscaled --tun=userspace-networking` needs no `/dev/net/tun` and exposes SOCKS5/HTTP proxy listeners; the same shape is available from sing-box's mixed inbound, `ssh -D`, or any SOCKS5 client library linked into the payload.

**P15 — Substitute the relay family.** `cloudflared` dials 7844 and nothing else, but ngrok, Pinggy, localtunnel, bore, zrok, Pangolin, Tailscale DERP, WARP/MASQUE and plain `ssh -R` all publish a service over 443 or 22. A detection keyed to 7844 or `argotunnel.com` sees none of them.

**P16 — Wear the wire format of something normal.** uTLS reproduces a browser's ClientHello; REALITY forwards unauthenticated probes to a real site; XHTTP pads 100–1000 bytes per request; hysteria2's salamander "scrambles every packet into seemingly random bytes"; AmneziaWG randomises WireGuard's fixed handshake; DoH/DoT carries a tunnel inside DNS-over-TLS on 443.

**P17 — Shape the traffic.** Pad, jitter post intervals, cap the rate, duty-cycle the workload, reconnect before a duration threshold. Any detector keyed to a constant rate, a fixed cadence or a long-lived flow is defeated by adding variance.

**P18 — Add cover traffic.** Synthetic ingress balances an out/in fan-out ratio; a busy loop defeats Oracle's idle reclamation; a decoy loopback listener that accepts defeats "connects but never accepts". The cover does not have to be useful, only plausible.

**P19 — Stay under the meter and the complaint threshold.** 1 GiB per 5-minute interval, 10 TB per month, and "nobody notices" all have a number. Choosing a quiet upstream — a private tracker, a small proxy pool, a customer who does not complain — means no threshold is ever crossed.

**P20 — UDP without `connect()`.** `sendto`/`sendmsg` on an unconnected socket creates no `connect` event and no `tcp_connect`; the connect/bind audit rules and the connect-vs-accept ratio both miss it, leaving only packet-level signatures.

**P21 — DNS as the allowed channel.** If egress is allow-listed to a resolver, `dnstt`/`iodine` reach the Electrosphere through that resolver's recursion. The allow-list is satisfied by design; the tunnel never touches a denied address.

**P22 — Abuse an allowed destination.** An allow-listed package index, a CDN, or a SaaS API is a relay: `git`/`pip` fetches carry data, webhooks and bot APIs (Telegram, Discord, SendGrid) carry mail and commands over 443 with no SMTP or SOCKS socket anywhere.

**P23 — Artifact hygiene.** Unset `GITHUB_ACTIONS`, clear IPython history, point `JUPYTER_RUNTIME_DIR` at `/tmp`, unlink the runtime files, backdate mtimes, remove the binary after `exec`.

**P24 — Flood the collector.** Overrun the audit backlog, force Zeek's `capture_loss` up, fill conntrack. Once the collector has dropped events, every downstream negative is void — and the flood can come from a legitimate process.

**P25 — Use a platform the checklist does not cover.** podman/containerd/nomad without the Docker CLI, Windows containers, a nested VM, static pods with no mirror object.

**P26 — Be the legitimate workload.** A real VPN, a real CI job, a real seedbox, a real restream, an employee's real remote desktop. No bypass is needed because there is no positive signal to evade. Every issue that ends this way carries `indistinguishable`.

## Detection index

The reading order, and the whole conclusion on one screen.

**`verdict`** is the issue's own finding, and it is a statement about *the detection*: `high` means the bypass is free or one ordinary design choice away, `medium` that it costs the squatter a specific but unremarkable move, `low` that the check holds against the population this document is about, `indistinguishable` that there is no residual signal at all, and `mixed` that two halves of one check land differently — the file says which. `—` marks the two documents that test no check.

**`keep`** is this document's recommendation, not the issue's: `run` means the check is worth running as written for the case it names, `triage` means it is a lead or answers a different question (exposure, entry, attribution) and should not be counted as squat coverage, `drop` means it should come out of a squatting checklist.

| # | detection | verdict | keep |
|---|---|---|---|
| [D01](D01-exe-path.md) | `/proc/<pid>/exe` path sweep | high | triage |
| [D02](D02-comm-kworker.md) | `comm` spoofed as a kernel thread | low | drop |
| [D03](D03-build-metadata.md) | Go/Rust build metadata in the binary | medium | triage |
| [D04](D04-preload.md) | `/etc/ld.so.preload`, any content | mixed | run |
| [D05](D05-persistence.md) | config mtimes + `lastlog` | medium | triage |
| [D06](D06-listeners.md) | listener and port table | high | drop |
| [D07](D07-established.md) | established sockets, `:7844` | medium | triage |
| [D08](D08-netns.md) | netns enumeration | medium | triage |
| [D09](D09-crosscheck.md) | netlink vs procfs disagreement | medium | triage |
| [D10](D10-attribution.md) | manual inode→pid join | medium | drop |
| [D11](D11-packetsock.md) | `AF_PACKET` sockets | low | drop |
| [D12](D12-ipaccounting.md) | systemd `IPEgressBytes` per unit | high | run |
| [D13](D13-nft-uid.md) | nft per-UID counter set | medium | triage |
| [D14](D14-conntrack.md) | conntrack asymmetry | medium | triage |
| [D15](D15-vnstat.md) | vnstat per-interface history | medium | run |
| [D16](D16-nethogs.md) | nethogs / bandwhich | mixed | triage |
| [D17](D17-audit-connect.md) | audit `socket`/`connect`/`bind` | high | drop |
| [D18](D18-audit-tun.md) | audit `TUNSETIFF` | medium | run |
| [D19](D19-audit-netns.md) | audit `clone`/`unshare`/`setns` | medium | triage |
| [D20](D20-audit-memfd.md) | audit `memfd_create` | low | drop |
| [D21](D21-audit-persist.md) | audit writes under `/etc` | medium | triage |
| [D22](D22-audit-kmod.md) | audit module load, `modules_disabled` | medium | triage |
| [D23](D23-audit-uring.md) | audit `io_uring` `openat` | high | triage |
| [D24](D24-audit-lost.md) | audit backlog and `lost` | medium | run |
| [D25](D25-ebpf-connect.md) | kprobe `tcp_connect` | medium-high | run |
| [D26](D26-ratio.md) | connect/accept ratio per `comm` | medium | drop |
| [D27](D27-ebpf-probes.md) | six-probe set | medium | triage |
| [D28](D28-tcplife.md) | `tcplife` / `bindsnoop` | mixed | triage |
| [D29](D29-tetragon.md) | Tetragon policy with `Sigkill` | low | run |
| [D30](D30-tracee.md) | Tracee signatures | low | drop |
| [D31](D31-ja4.md) | JA3/JA4 fingerprinting | indistinguishable | drop |
| [D32](D32-udp-sigs.md) | QUIC / WireGuard / STUN constants | medium | triage |
| [D33](D33-bittorrent.md) | BitTorrent, DHT, tracker magic | medium | triage |
| [D34](D34-mse-shape.md) | MSE/PE first-flight shape | medium | drop |
| [D35](D35-socks-stratum.md) | SOCKS5 and stratum constants | medium | drop |
| [D36](D36-dns.md) | DNS tunnelling entropy | medium | triage |
| [D37](D37-rita.md) | RITA / ntopng | medium | drop |
| [D38](D38-zeek-ws-capture.md) | Zeek WS shape + `capture_loss` | mixed | triage |
| [D39](D39-tor.md) | Tor exit lists | medium | run |
| [D40](D40-shodan.md) | Shodan InternetDB / Censys | mixed | triage |
| [D41](D41-ct-pdns.md) | `crt.sh` + passive DNS | mixed | triage |
| [D42](D42-rdap.md) | RDAP / PTR / ASN | low | run |
| [D43](D43-dnsbl.md) | Spamhaus, SpamCop, AbuseIPDB | medium | triage |
| [D44](D44-honeypots.md) | `endlessh` / Cowrie | low | triage |
| [D45](D45-tripwires.md) | canarytokens, `chattr +i`, `atime` | medium | triage |
| [D46](D46-systemd-path.md) | `systemd.path` tripwire | high | run |
| [D47](D47-audit-readwatch.md) | audit read watch | medium | run |
| [D48](D48-jupyter-artifacts.md) | notebook artifacts | medium | triage |
| [D49](D49-notebook-controls.md) | notebook cgroup / allow-list controls | medium-high | run |
| [D50](D50-container-apis.md) | container API exposure | medium | run |
| [D51](D51-k8s-objects.md) | Kubernetes object queries | medium | triage |
| [D52](D52-torrent-host.md) | seedbox CPU/egress/disk signature | medium | drop |
| [D53](D53-ci-runners.md) | CI runner checks | mixed | triage |
| [D54](D54-remote-desktop.md) | RDP/VNC listeners + event logs | medium | triage |
| [D55](D55-relays.md) | relay ports and size ratios | medium | run |
| [D56](D56-streaming.md) | restream counters and logs | medium | triage |
| [D57](D57-provider.md) | provider billing and abuse reports | medium | run |
| [D58](D58-censor.md) | the censor-side classifier | — | n/a |
| [D59](D59-gaps.md) | channels with no detection at all | — | n/a |

34 of the 59 verdicts are `medium`, so the table says the same thing on most rows: the check fires on a default deployment and is erased by one flag. Eight are `high` or `medium-high` — sound the early alarm on those first. Seven are `low`, and one of those (D29) is `low` in the *other* direction: it is the best of the tier, and the number describes how much a bypass would cost rather than how little the check is worth.

## What actually survives

Read as a whole, the sweep leaves eight things standing, and one conclusion about the rest.

1. **Whole-host egress, reconciled against declared purpose.** Sum every cgroup under the host and subtract the sum of what each unit is supposed to be doing. D12's per-unit sample is gameable (P12); the whole-host sum is not, because a scope the attacker creates still counts. Getting past it needs kernel-level tampering rather than a configuration flag, which is no longer a cheap bypass.
2. **The provider's view.** Billing, flow logs and abuse reports (D57) are the only detector not running on the host being detected. Everything the guest can edit — counters, processes, sockets, logs — is one root command from a lie (P04); a bill is not.
3. **In-kernel enforcement, not description.** Tetragon with a policy that covers TCP *and* UDP and kills on match (D29) is the only on-host tier that removes capability rather than producing evidence. It still needs a policy that a human can read, which is why the data-shaped policy is worth more than the probe.
4. **The eBPF record, kept for after.** `tcp_connect` plus a UDP probe (D25, D27) is the best on-host history available, and it survives inside a container because a kprobe is global to the kernel. Its blind spots (pre-existing flows, fd passing, io_uring) are the design choices a squatter who has read this makes.
5. **One egress allow-list, without a resolver.** `IPAddressDeny=any` plus an allow-list is the single control that removes a capability (D49). The instant DNS is allowed — which is what ships — it becomes a bandwidth cap on the tunnel.
6. **Audit, filtered and integrity-checked.** The `io_uring` filter extended to `connect`/`bind`/`sendmsg` (D23) and `lost` read at the moment of alert rather than on a sweep (D24). Audit's syscall rules lose to io_uring and `clone3`; its filters are the part that holds.
7. **Cheap exact-format checks, for triage.** `/etc/ld.so.preload` content (D04), a `TUNSETIFF` rule (D18), the relay port table (D55), the container API port list (D50). These are worth running because they cost nothing and never lie — they just do not fire on a careful squat.
8. **Tripwires, armed properly.** `systemd.path` with both trigger limits at 0 and `OnFailure=` wired (D46), and audit read watches where every access must be seen (D47). Both are defeated by root; both are free.

**And the conclusion about everything else.** No on-host artifact separates a squat from a workload the operator's own tenant runs on purpose (P26, D31, D52, D58). A host provisioned for a purpose and then used for that purpose produces no anomaly, and every check here that is not measuring *volume against a declared intent* is measuring the packaging. The corpus's own structure says this and then spends its length on packaging — which is D59's finding, and the reason the list above is ordered the way it is: the further a check sits from the guest, the less it can be argued with.
