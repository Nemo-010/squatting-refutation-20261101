Part of the refutation sweep (#1). Primitives are defined there.

**Claim under test.** `/proc/<pid>/exe` is the only unspoofable identity, so sweeping it for `/tmp`, `/var/tmp`, `/dev/shm`, `/run`, `~/.cache`, `(deleted)` and `/memfd:` finds a squatter (`checks/triage.sh`, `research/proxy-tunnel-host-artifacts.md` §1.2).

**P01 — the path list is a whitelist of staging directories, not of writability.** `$HOME/.local/bin` and `$HOME/.local/lib/python3.*/site-packages` are writable at any uid and match none of the patterns; a `pip install --user` console script or a `PRoot`/`udocker` rootfs under `$HOME` runs from a path the sweep never looks at. `/opt`, `/srv`, `/usr/local/lib` and `/var/lib/<name>` are root-owned 0755 on a stock distro, so those need the root the installer already has.

**The container path matches nothing.** A container process is visible in host `/proc`, but its `exe` resolves into the overlay — `/var/lib/docker/overlay2/<id>/merged/usr/local/bin/...` or `/var/lib/containers/storage/overlay/...`. None of the seven globs match, so the deployment the corpus calls the delivery vehicle of choice reports as clean.

**P02 — no executable at all.** `python3 -c`, `node -e`, a `dlopen`ed `.so` inside an existing daemon. There is no `/proc/<pid>/exe` to read for the payload, only the host interpreter's, which is `legitimate`.

**Unprivileged blind spot.** `readlink` on another user's `/proc/<pid>/exe` fails the ptrace access check, and both `triage.sh` and the research snippet `continue` on failure. `triage.sh` does print a "not root" note first, but the output is the same shape as a clean host: an EACCES-suppressed sweep and an idle machine are indistinguishable in the result. Count the `EACCES` returns separately, or the check cannot tell "nothing there" from "not allowed to look".

**P03 / P04.** A preload hook can rewrite the `readlink` result; an LKM can point `exe` elsewhere or hide the dentry. `prctl(PR_SET_MM_EXE_FILE)` repoints it too, but needs `CAP_SYS_RESOURCE`.

**Verdict: high.** P01 needs a home directory, P02 needs nothing, and both are the normal way software runs. The check catches a dropper that follows the 2023 campaign pattern (stage in `/dev/shm`, `chmod`, run, unlink) and nothing more careful. When the payload is `apt`/`pip`/`npm`-installed under a normal prefix it is also `indistinguishable` from software the operator installed on purpose — the path is not evidence of intent.
