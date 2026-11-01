Part of the refutation sweep (#1).

**Claim under test.** Netlink `sock_diag` and procfs are two independent data sources; a disagreement between them is the finding. `PROC_NET_TCP=/proc/net/tcp ss -Htan` forces the same binary through procfs (`checks/cross-check.sh`, `research/rootkits-that-hide-the-squat.md` §4).

**They are not independent.** `tcp_diag_dump()` and `tcp4_seq_show()` are two accessors over the same in-kernel socket hash table. They differ in *code path*, not in *source of truth*. P04 (LKM/eBPF hooking the table or the seq_file output) defeats both identically, and the check reports agreement.

**P03 hooks both sides.** `fopen("/proc/net/tcp")` covers the procfs read; `sendmsg`/`recvmsg` covers the netlink request/response. A hook that filters the `inet_diag` reply by port produces a consistent, smaller world in both views. The corpus lists "a sendmsg/recvmsg hook (none listed)" as the defeat — "none listed in the published specimens" is a statement about 2020–2023 malware, not about capability.

**Both views are scoped to one netns.** `ss -Htan` and `/proc/net/tcp` see the current namespace only. A container on its own netns is empty in both, so the check reports agreement — a clean bill of health for precisely the deployment the corpus says is default. Agreement is evidence *within this netns* and nothing more; it has to be paired with the D08 enumeration or a host-wide counter before it can be read as clean.

**The comparison is counts, so it races.** Sockets open and close between the two `ss` invocations; the script re-runs "immediately" and otherwise tells the operator to investigate. P24 turns that into a false-positive generator: any process opening/closing sockets at a moderate rate produces disagreement on demand.

**The `PROC_NET_TCP` trap is real but self-inflicted.** `PROC_NET_TCP=1` opens a file named `1` and yields a guaranteed disagreement; the corpus documents this correctly, which is the one part of this check that is solid.

**Verdict: medium.** It reliably catches the 2020-era preload rootkit that filters only `fopen("/proc/net/tcp")`. It does not establish truth, because no view reachable by `ss` is outside the loader's reach when the loader is compromised. The only genuinely independent tiebreaker in the corpus is the kernel-side counter, and that is a different check (D12).
