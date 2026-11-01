Part of the refutation sweep (#1).

**Claim under test.** `update @suids { meta skuid counter }` in one dynamic set gives per-UID egress accounting with no daemon, and survives a renamed binary (`checks/accounting.nft`).

**P10 — the match needs the socket's netns to differ.** `nft_meta_get_eval_skugid()` requires `skb_to_full_sk()`, `sk_fullsock()` and a matching netns. A relay in a container with a **private** netns never matches. A `--network=host` container or a `hostNetwork` pod shares the host netns and counts normally — which matters, because host networking is how proxyware images are usually deployed to get the "unused bandwidth" un-NAT'd.

**Root collapses to one bucket.** A squatter with root — the normal case for the exploits that install proxyware — sends as UID 0. On any host, UID 0 moves package downloads, container image pulls and log shipping. The counter for UID 0 is unreadable as evidence, and the operator has no baseline that separates 40 GB of `apt` from 40 GB of relay.

**Attribute to a UID that is supposed to talk.** If the payload runs as `www-data` or `postgres`, the counter shows traffic from a service whose job includes network I/O. The counter is *correct* and the conclusion is wrong.

**P04 / P09.** `nft flush ruleset` from a root that holds `CAP_NET_ADMIN` in the *host* namespace removes the table — and a privileged pod or `--cap-add NET_ADMIN` is exactly that. A `CAP_NET_ADMIN` in a mere user namespace only governs its own netns, which is P10 again. A bind-mounted `/etc/nftables.conf` (P09) survives a config audit.

**Forwarded traffic.** The rule sits in `output`. Anything routed — a container's NAT, a TUN forwarding packets, a bridged VM — has no originating socket and is not counted. This is the same blind spot as `-m owner`, restated.

**Verdict: medium.** As a live gauge of which UIDs move bytes it is useful; as an *inventory* it is not, because the set and its counters are kernel-resident only and vanish on `flush ruleset` (the standard preamble of `/etc/nftables.conf`), on `nft reset counters`, and on reboot unless something samples and persists them. As attribution it answers "which UID", which on a compromised host is the one fact the attacker already controls.
