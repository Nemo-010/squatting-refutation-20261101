Part of the refutation sweep (#1).

**Claim under test.** `ss -lntp` for 2375/2376/2379/2380/8983/10255, read `/var/lib/kubelet/config.yaml`, `curl 127.0.0.1:2375/version`, and the hex-port fallback in `/proc/net/tcp` (`research/kubernetes-and-container-squatting.md` §10).

**The Unix socket is the real API and it is not a port.** `/var/run/docker.sock`, `/run/containerd/containerd.sock`, `/run/crio/crio.sock` are the defaults. Anyone in the `docker` group is root-equivalent through them, and the TCP check finds nothing. The corpus checks the daemon's *TCP* listener — the exposure that a hardening guide would call the mistake — not the socket that is always there.

**Rootless podman has a socket too.** `/run/user/<uid>/podman/podman.sock` gives the tenant container control with no root and no Docker CLI (P25). It appears in none of the checks.

**10250 with a valid token is not an anomaly.** The corpus's kubelet check targets anonymous auth and the read-only port. An attacker holding a service-account token — or the node's own client certificate at `/var/lib/kubelet/pki/` — uses the authenticated port, and every log line says a legitimate identity did it.

**IPv6-only bind.** The checks are `ss -lntp` (which is both families, fine) and the `/proc/net/tcp` hex fallback, which is v4 only. A daemon bound only on `::` misses the fallback; the corpus notes the analogous trap for RDP and not here.

**Verdict: medium.** The port list is right for a misconfigured host and the config-file check is the best single item in this slice. Both are about *exposure to the Electrosphere*, and the squat the corpus describes usually arrives with credentials, which makes exposure irrelevant.
