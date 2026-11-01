Part of the refutation sweep (#1).

**Claim under test.** Torrent daemons are found by process name and port table, and a seedbox is distinguished by the three-way signature: low CPU, high one-way egress, high disk read (`/proc/<pid>/io`) and a growing disk (`research/torrent-seedbox-detection.md` §4.3, §5).

**P13 — the name is a string.** `qbittorrent-nox` copied to `systemd-helper` matches no name rule. The binary is C++, so there is no Go module path to recover; `strings` still finds literals, which is why the payload is a *container image* with the name in the tag.

**P10 — the container is the deployment.** A `linuxserver/qbittorrent` container has a familiar image name, and the process inside it is not in the host's `ps` output at all.

**The disk signature is weaker than it looks.** `read_bytes` counts storage reads. On a seedbox with enough RAM the working set is in page cache, so `rchar` rises and `read_bytes` does not — the "high disk read" column is a cache-size measurement. A client that deletes completed torrents keeps the disk flat. Neither number is required by the protocol.

**The egress is the only durable signal.** The corpus says this, and then lists the disk columns as the discriminator. A private-tracker seeder whose egress is metered is indistinguishable from a large legitimate backup or a mirror (P26); the corpus's own §3 table concedes that reputation and complaints do not apply, leaving only the metre.

**Verdict: medium.** Name and port are trivially changed, the disk columns are page-cache dependent, and the one signal that survives — sustained one-way egress — is a byte counter, which is D12/D15 with all of their bypasses. For a private seedbox on a provider that does not care about torrents (the corpus quotes DigitalOcean's AUP never mentioning them), this is `indistinguishable` from a legitimate mirror.
