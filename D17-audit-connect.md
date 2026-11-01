Part of the refutation sweep (#1).

**Claim under test.** The audit rules on `socket`, `connect` and `bind` record every outbound connection, split by address family because `saddr_fam` is only populated for bind/connect/sendto (`checks/50-proxyjack.rules`, `research/auditd-proc-detection.md` §3).

**P05 — io_uring.** `IORING_OP_CONNECT` does not enter through the syscall table; the `exit` filter never evaluates it. It is auditable, but only by a rule on the *io_uring* filter naming `-S connect`, which this ruleset does not have (see D23). `auditctl` does map `-S connect` to the opcode via `audit_name_to_uringop`, so the gap is the rule, not the kernel.

**P20 — `sendto`/`sendmsg` are not in the rule.** The rules match `connect` and `bind` only. A UDP tunnel that calls `sendto()` on an unconnected socket produces no `net_conn` event at all. This is the normal way to write a QUIC/hysteria/`dnstt` client.

**P06 / P07 — no `connect()` in the window.** The fd is connected elsewhere (a helper process, or at boot) and passed in, or inherited from socket activation. The event exists — attributed to a pid that has exited — or predates the rules.

**b32 is missing.** The corpus notes the need for `arch=b32` twins. A 32-bit payload (an old static binary, a musl i386 build) has different syscall numbers and matches neither rule.

**Other families.** `AF_VSOCK` (40), `AF_XDP` (44) and `AF_PACKET` (17) sockets are not in the `a0=2`/`a0=10` rules. A relay to a co-resident VM or over a userland stack never matches.

**Containers.** The record carries no tenant identity: `pid` is the host pid and `loginuid` is typically unset. Attribution has to be resolved at triage time from `/proc/<pid>/cgroup` or the netns inode, and an event whose pid has already exited resolves to nothing.

**Verdict: high.** Three of the four bypasses need no privilege beyond writing a different socket call. The ruleset detects a developer who wrote the naive `socket()`+`connect()` loop, which is not the population this document is about.
