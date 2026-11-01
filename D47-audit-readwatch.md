Part of the refutation sweep (#1).

**Claim under test.** `chattr +i` and `atime` are not read tripwires; use auditd or inotify when every access must be seen (`research/external-vantage-and-honeypots.md` §6.3).

**P09 — bind-mount over the watched tree.** A watch is an inode mark. `mount --bind` a different directory over the canary's directory and reads land on an inode the rule never saw. Root, `CAP_SYS_ADMIN`, one command.

**`perm=r` keys on the open flags, not on `read`.** auditctl(8): the read and write syscalls are omitted from the set because they would overwhelm the log; instead "the open flags are looked at to see what permission was requested". A plain `mmap` of the canary therefore still produces the `open` event. The no-privilege bypass is to receive an already-open fd — `SCM_RIGHTS`, an inherited fd, or one opened before the rule loaded — and read or `mmap` that.

**io_uring does not help on its own.** `READV`/`READ` are `audit_skip`, but the read permission is judged at `open`, and `IORING_OP_OPENAT` is audit-visible. An io_uring read of a normally opened canary is therefore caught at the open; only an io_uring *open* outside audit's view, or a passed-in fd, escapes.

**inotify has a queue, and the queue overflows.** `IN_Q_OVERFLOW` drops events by design when a reader falls behind. Flooding a watched directory — a build, a log directory, a package upgrade — is both a normal workload and a way to lose the one event that mattered.

**Verdict: medium.** Audit read watches are the right tool and they work against the case the corpus describes (someone opening a stored credential). They are defeated by any of three ordinary mechanisms, and the strongest one (`mmap`) needs no privilege at all.
