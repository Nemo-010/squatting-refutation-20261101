Part of the refutation sweep (#1).

**Claim under test.** `-F dir=/etc -F perm=wa` replaces the broken `/etc/ld.so.preload` watch, and `dir=` watches on `/etc/cron.d`, `/var/spool/cron` and `/etc/systemd/system` catch persistence (`checks/50-proxyjack.rules`).

**P09 — bind-mount over the watched tree.** Watches are inode marks on the original directory. `mount --bind /home/x/etc /etc/systemd/system` puts a different tree at the watched path; writes land on inodes the rule has never seen. Root with `CAP_SYS_ADMIN` is enough, and root is what the installer has.

**The list is three directories.** `~/.config/systemd/user`, `/usr/lib/systemd/system`, `/etc/profile.d`, `~/.bashrc`, `~/.ssh/rc`, `/etc/crontab`, `at`, `pam.d`, `ld.so.conf.d`, `supervisor`, a container restart policy — none are watched. A user-level unit needs no root at all.

**Mounts made after the rule load are the gap.** auditctl(8) tags child mounts that exist when the watch is issued, so a container overlay already in place is covered. A `mount --bind` performed *after* the ruleset loads is not, unless `-q mount-point,subtree` is added — and it is not. P09 is that move.

**io_uring writes.** `-S openat,openat2` on the uring filter covers the open; a `write()` through io_uring is not audited (those opcodes are marked `audit_skip`). Creation still needs an open, so this is a partial gap — but a rename or a `linkat` of an already-open temp file is not covered either.

**Every persistence rule is b64-only.** The `etc_write` and `persist` rules carry `arch=b64`, so a compat (i386) writer's `open`/`renameat` into `/etc`, `/etc/cron.d` or `/etc/systemd/system` matches nothing — the same b32 hole D17 raises for sockets.

**The key is noisy by design.** `/etc` changes on every package upgrade. `etc_write` will produce thousands of events a day on a normal host, which is precisely how an operator learns to filter the key.

**Verdict: medium.** It is better than a `-w` on a file that may not exist, and worse than the problem requires. The watched set is the author's memory of the 2019 installer layout, and the most common persistence on a modern host (`systemd --user`) is not in it.
