Part of the refutation sweep (#1).

**Claim under test.** `ss -p` resolves owners through `opendir("/proc")` and `readlink("/proc/PID/fd/N")`, so it is hookable; the fix is a manual join from `ss -tne` inode to `/proc/*/fd/*`, bounded by `/proc/loadavg` field 5 (`research/rootkits-that-hide-the-squat.md` §4).

**The win is smaller than claimed.** The join replaces `readdir("/proc")` and nothing else: it still calls `opendir`/`readdir` on `/proc/<pid>/fd`, and `readlink`/`stat` on the entries. A hook that rewrites `readlink` to return `anon_inode:[eventfd]` for one inode hides the process from the manual join exactly as it hides it from `ss -p` — both call the same libc function.

**P06 — attribute to the holder, not the author.** Pass the connected fd with `SCM_RIGHTS`, close your copy, exit. The only process with `socket:[inode]` in its fd table is the receiving one, which is chosen to look legitimate. The join is now *correct* and *misleading*.

**P12 / P10.** A process in another netns has no socket in this view; a process in another PID namespace still appears in host `/proc`, but a VM does not.

**P04.** If the kernel hides the pid, `stat` fails and the fd directory is empty.

**The `last pid` bound is racy and expensive.** `/proc/loadavg` field 5 is the last pid *created*, not the highest pid alive. A join that starts at 2 and walks to it is O(pid_max) on a long-lived host, and a pid reused after the bound is missed — the check has a silent upper edge.

**Verdict: medium.** It defeats a hook that filters the `/proc` *directory listing* and nothing more. Against a hook on `readlink`/`stat` it is no better than `ss -p`, and against fd passing it produces a confident, wrong attribution.
