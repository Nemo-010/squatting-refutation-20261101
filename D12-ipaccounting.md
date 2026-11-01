Part of the refutation sweep (#1).

**Claim under test.** `IPAccounting=yes` gives per-unit `IPEgressBytes`; it is the cheapest per-service signal and the tier "that pays" (`checks/egress-watch.sh`, `research/per-user-network-accounting.md` §2).

**P12 — spawn outside the unit.** The counter belongs to a cgroup. `systemd-run --scope` from a non-root user creates the scope under the *user* manager, which the system manager never lists at all; `systemd --user` does the same. The notebook's `IPEgressBytes` stays flat while the bytes leave. Scopes only help when they are system units that already carry `IPAccounting=yes`, and a scope without it reports `UINT64_MAX`, not a delta.

**P10 — the container may not even be a scope.** Docker's default cgroupfs driver puts a container at `/sys/fs/cgroup/docker/<id>` with no unit at all, so there is no `IPEgressBytes` to read and nothing for `list-units` to enumerate. Under the systemd driver it is a sibling scope, and per-unit accounting still does not aggregate across siblings.

**Forwarded traffic is not accounted.** A TUN (WireGuard, sing-box `tun`) or a NAT'd sibling container egresses as forwarded skbs with no socket, so the `cgroup/skb` hook never fires: the counter stays at 0 and no amount of scope enumeration fixes it.

**Restart to zero the delta.** The counter resets when the unit restarts, and `egress-watch.sh` explicitly discards a negative delta as a restart. A workload that exits every few minutes (or has `Restart=always` and a supervisor that cycles it) never presents a delta above the threshold. The alert condition is "delta > 1 GiB per 5 min"; the bypass is "be a new unit often".

**P10.** A container's processes are in a sibling cgroup, or in no cgroup unit at all; per-unit accounting does not aggregate across scopes, and the corpus's "not recursive" caveat is about slices — the same non-recursion applies here.

**P19.** 1 GiB per 5 minutes is ~28 Mbit/s sustained. A tunnel at 10 Mbit/s is inside a free tier's terms and under the alert forever.

**Counter coverage.** The hook is IPv4/IPv6 `cgroup/skb`. Raw `AF_PACKET` frames and non-IP traffic are not counted; `MSG_ZEROCOPY`/GSO paths are the documented undercount.

**Verdict: high.** The strongest *idea* in the corpus — measure bytes at the cgroup rather than inspect flows — is implemented as a per-unit sample with a fixed threshold, and both properties are attacker-controlled. Summing all scopes for a whole-host total and comparing against the sum of declared purposes would be much harder to game; that is not what the script does.
