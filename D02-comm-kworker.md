Part of the refutation sweep (#1).

**Claim under test.** A process whose `comm` is `kworker/0:1` and whose `/proc/<pid>/exe` resolves is always a lie, because a real kernel thread has no user address space (`checks/triage.sh`, `research/proxy-tunnel-host-artifacts.md` §1.1).

The claim itself is correct. Note that `comm` never contains brackets — `ps` draws them for kernel threads — so a rule or audit key written for `[kworker/0:1]` can never match.

**P13 — pick a different name.** The check fires only on seven globs. `kcompactd0`, `ksmd`, `khugepaged`, `jbd2/sda0-8`, `kauditd`, `irq/24-nvme0q1` and `scsi_eh_0` all render as kernel threads and pass the case list — the impersonation survives without leaving the namespace. And `comm` is a free 16-byte string: `sshd`, `dbus-daemon`, `python3`, `postgres` pass too. The corpus's own specimen set `argv[0]` to `[kworker/1:0H]` (P2PInfect), which is argv spoofing, not `comm` spoofing; nothing forces either.

**Truncation helps the squatter.** `comm` is 16 bytes including the NUL, so a long descriptive name is indistinguishable from its prefix — and `ps` output is what most operators actually read.

**P03 / P04.** `readdir` hooks remove the process from `ps`; an LKM removes it from `getdents` and `kill`. Either way the name never gets compared.

**The cheap correct discriminator is structural.** Kernel threads have `PPid 2` in `/proc/<pid>/status` and an empty `/proc/<pid>/maps`. Testing those instead of a name list catches every impersonation including the seven names the globs miss — and the corpus does not use them here.

**Verdict: low.** This is a tautology detector: it catches only a process that chose one of seven names *and* kept a resolvable `exe`. Useful as a one-line filter, worthless as a control. The stronger version of the same idea — compare `comm` against the binary's real identity for every process, not just kernel-thread names — is not in the corpus and would still lose to P13 (naming a binary `sshd`) unless `exe`'s inode is checked against the package database.
