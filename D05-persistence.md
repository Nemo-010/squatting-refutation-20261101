Part of the refutation sweep (#1).

**Claim under test.** Recent writes to `/etc/cron.d`, `/var/spool/cron`, `/etc/systemd/system` and `/etc/rc.local` (last 7 days) plus `lastlog` entries that never logged in reveal the installation (`checks/triage.sh`).

**The list is short.** `~/.config/systemd/user`, `/usr/lib/systemd/system`, `/etc/profile.d`, `~/.bashrc`, `~/.ssh/rc`, `/etc/crontab`, PAM config, `ld.so.conf.d`, `supervisord`, a Docker restart policy, a k8s DaemonSet, and `/etc/ld.so.preload` itself are all persistence surfaces the `find` does not stat. A `systemd --user` unit survives reboot with no root, given lingering — and the artefact that proves lingering is `/var/lib/systemd/linger/<user>`, which is not checked either.

**mtime is attacker-controlled.** `find -newermt` compares mtime. `touch -d '2 years ago'`, `utimensat`, or writing the file through a tool that restores timestamps defeats it. Nothing in the check compares mtime against ctime or inode generation, which would be much harder to forge.

**Live off the land, then fix the clock.** Append one line to an existing cron file, or enable an existing unit — both update mtime, so both are caught by the recency check unless the timestamp is reset afterwards. The two moves are one sequence, not two alternatives, and the corpus describes only the first.

**`lastlog` misses the accounts that matter.** Squatting usually reuses an account that has logged in before. `lastlog` also does not update for `su`, for a non-login shell, or for a process started by a supervisor — so "never logged in" is a narrow net, and an actor who wants a clean record uses an account with an existing login.

**P23.** Config lives in memory, in an environment variable, or is re-fetched from a subscription URL at start; the on-disk artifact is a 3-line wrapper that looks like any other service.

**Verdict: medium.** It is a snapshot of one directory list at one moment. Its value is catching a careless `curl | bash` installer, and the corpus's own headline example (`mack-a/v2ray-agent`) is careless. It does not survive a single `touch`, and it cannot see user-level persistence at all.
