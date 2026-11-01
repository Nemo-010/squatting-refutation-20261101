Part of the refutation sweep (#1).

**Claim under test.** nethogs/bandwhich attribute bandwidth per process, and nethogs's explicit "unknown TCP" bucket is itself a signal: "on an idle server it should be near zero" (`research/per-user-network-accounting.md` §4).

**The bucket is not near zero on any modern host.** Container traffic, sockets in another netns, short-lived processes and anything whose inode the tool loses land in "unknown". On a host running Docker or a CNI, the baseline is large, which makes "unknown > 0" useless as an alert and makes the number itself uninterpretable.

**P10 / P12.** The tool reads the host netns and host `/proc`. A container's traffic is attributed to `docker-proxy`, `containerd-shim` or nothing.

**P06.** The fd holder shows, not the author.

**P04.** Both tools resolve through `/proc`; a `readdir` hook changes what they attribute, and the "unknown" bucket absorbs the difference — the failure mode is silent by construction.

**Permissions.** nethogs needs `cap_net_admin,cap_net_raw,cap_dac_read_search,cap_sys_ptrace`; without them `pcap_open_live` fails and it exits with an error rather than degrading. bandwhich needs four capabilities and loses per-process resolution without them. Either way the failure is loud, not a D15-style aggregate — which is the one good property here.

**Verdict: low as detection, high as triage.** These are the right tools for "what is this process doing right now" once something is already suspected. They are not a control: they have no history, they need privileges, and their failure mode (unknown/unattributed) is indistinguishable from their success mode (attributed to a shim).
