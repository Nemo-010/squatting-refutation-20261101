Part of the refutation sweep (#1).

**Claim under test.** Enumerate `readlink /proc/*/ns/net | sort -u`, `nsenter -t <pid> -n -- ss`, and `docker inspect` for `NetworkMode=host` / `Privileged` (`checks/triage.sh`, `research/proxy-tunnel-host-artifacts.md` §2.3).

**P25 — the runtime is not necessarily Docker.** podman, containerd/nerdctl, CRI-O, LXC/Incus, systemd-nspawn, nomad and Firecracker all create netns without a `docker` binary. `docker` absent prints "docker not present" and the check stops; the namespaces are still there and the enumeration still lists the inodes, but nothing tells the operator what any of them is.

**A netns does not need a live process.** `ip netns add foo` persists the namespace as an `nsfs` bind mount at `/var/run/netns/foo` after the creating process exits. `/proc/*/ns/net` never lists it, so the enumeration returns zero rather than "one I did not create" — a stronger evasion than the short-lived case below. Compare against `/var/run/netns/*` and `findmnt -t nsfs` too.

**Nested namespaces.** `/proc/*/ns/net` lists every netns that has a live process — including one inside a container. What it cannot do is say which are legitimate. On a host running a CNI, the count is dozens before anything is squatted.

**Both steps need privilege.** `readlink /proc/<pid>/ns/net` passes through the same ptrace access check as `exe`, so a non-root run silently misses other users' namespaces, and `nsenter`/`setns` needs `CAP_SYS_ADMIN` in the namespace's owning user namespace. On a hardened host or from a container, the count is the only thing that survives.

**P04.** An LKM can hide namespace symlinks, or the process holding them.

**Short-lived namespaces.** The sample is instantaneous. A pod that lives for a job and exits leaves nothing to enumerate; if it also does the work, there is no persistent netns to find.

**Verdict: medium.** Enumeration is sound and cheap; the analysis is the weak part. "Distinct netns inodes: 47" on a Kubernetes node is not a finding, and the check gives the operator no way to tell which of the 47 is the one they did not create. The missing comparison is against the orchestrator's object list, not against zero.
