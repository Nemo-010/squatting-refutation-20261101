Part of the refutation sweep (#1).

**Claim under test.** With `nf_conntrack_acct=1`, per-flow byte counters show a tunnel as asymmetry: "tens of GB in the reply→original direction while the original direction is nearly silent" (`research/per-user-network-accounting.md` §3).

**Enabling the sysctl does not reach the flow you are hunting.** The accounting extension is added when a conntrack entry is created, and only if the sysctl is already on (`nf_ct_acct_ext_add()` returns NULL when it is 0). Turning `nf_conntrack_acct=1` on now leaves every established flow — including the long-lived tunnel — with no `packets=`/`bytes=` at all. The counter has to be on before the flows start.

**That signature is a download.** reply→original large, original small is what every client download looks like — `apt`, `docker pull`, a browser, an off-site backup restore. On a proxyjacked box serving *other people's downloads*, the same shape appears for a completely different reason. The heuristic does not separate the two, and "a legitimate client upload is asymmetric the other way" only distinguishes upload from download, not relay from client.

**A relay is often symmetric.** Third-party TCP through the box produces bytes in both directions, roughly balanced over time. Balanced is also the normal profile of any session-heavy service; the tunnel the corpus is looking for does not have the shape it describes.

**The distinguishing feature is not in conntrack.** What makes a relay a relay is that many unrelated clients arrive through one upstream flow. Conntrack sees one flow to the CDN. The fan-out exists at the CDN, not here.

**Off by default.** `nf_conntrack_acct` ships 0. An operator who has not enabled it has no counters, and nothing in the squat's behaviour depends on the setting.

**P10 / P24.** A container's flows are in its own netns; on a busy host the table fills and drops, which is both a coverage failure and a lever the attacker can pull from a legitimate-looking process.

**Verdict: medium.** Flow accounting is good for "which peer is this host talking to, and how much", which is why the triage use of `conntrack -L -o extended` is sound. The asymmetry inference is not: it is a download detector wearing a tunnel detector's name.
