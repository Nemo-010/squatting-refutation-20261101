Part of the refutation sweep (#1).

**Claim under test.** `/etc/ld.so.preload` is the load-bearing file: any content is a finding, a deleted preload object shows as `(deleted)` in `/proc/<pid>/maps`, and Kinsing's `EPERM`-instead-of-`ENOENT` is a fingerprint (`research/rootkits-that-hide-the-squat.md` §3, §5).

**P03 without the file.** `LD_PRELOAD` in `~/.bashrc` is per-user and never touches `/etc`. Every detection above keys on `/etc/ld.so.preload`; a userland hook installed this way leaves the file absent and the "any content is a finding" check clean.

**Keep the object on disk.** The `(deleted)` marker only appears after `unlink`. A preload object at `/usr/lib/x86_64-linux-gnu/libnss_*.so.2`-looking path, never deleted, is an ordinary mapped library in the maps sweep.

**Filter the reader.** The rootkit's job is to lie to `open`/`read`; `/proc/<pid>/maps` is read through exactly those calls. A hooked `read` can drop the library's lines, which also defeats the rkhunter `shared_libs` test that reads the same file through the same libc.

**Reach the checker through its own environment.** A systemd drop-in — `/etc/systemd/system/egress-watch.service.d/hook.conf` with `Environment=LD_PRELOAD=...`, or `DefaultEnvironment=` in `/etc/systemd/system.conf` — preloads the hook into the monitoring unit without touching `/etc/ld.so.preload`. `/etc/environment` through `pam_env` does the same for every login. Both are writable by root, which the rootkit has.

**P04.** An LKM needs no preload file at all. `kernel.modules_disabled=1` is a one-way runtime switch: it can be set at any time and cannot be set back, so on a host where the operator has already set it the LKM class is closed — and where they have not, the attacker sets the preload instead, which needs no module.

**The parser gap is a false-positive source, not an evasion.** glibc blanks `#` to end-of-line and splits on `:`; rkhunter strips leading `#` and word-splits, so a colon list or a trailing comment makes the checker complain about a token that is not a path. Every name the loader actually loads is still reported — and `triage.sh` does not parse the file at all, it `cat`s it. So this is noise for a human reader, not a hole.

**Verdict: high for the specific specimen, low for the technique.** The check finds Kinsing-era installs. It does not find a per-user preload, a non-deleted object, or a kernel-level rootkit — and each of those is a strictly easier deployment for someone who has read this document. `indistinguishable` does not apply here: a preload object in a server's library path is a genuine anomaly, just one this check does not reach.
