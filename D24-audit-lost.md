Part of the refutation sweep (#1).

**Claim under test.** Silent tail-drop is the failure that makes dashboards lie; `-b 8192`, `--backlog_wait_time 60000`, `-r 0` and reading `auditctl -s` → `lost` are the tuning that keeps the evidence (`research/auditd-proc-detection.md` §4).

**P24 — overflow on purpose.** The queue is finite. A process the operator already runs — a torrent client's connect storm, a build, a scan — produces enough events to overflow it, and `rate_limit` above 0 *drops* and consults the failure flag. The corpus's own justification for `-b 8192` is exactly this storm, which means the attacker's lever is a normal workload.

**`lost` is read by a cron that logs.** `audit-sweep.sh` writes a warning to syslog every five minutes. Nobody is paged on it, and an attacker only needs the drops to coincide with the tunnel's traffic.

**`-e 2` does not protect availability.** It prevents rule changes, not queue overflow. `overflow_action=suspend|single|halt` would be louder, and the corpus discusses it — but the shipped config does not set it.

**The userspace queue drops silently.** `q_depth` (default 2000) bounds auditd's queue to its plugins; a slow plugin — and the shipped design runs a script per line — drops events there, and those losses never appear in `lost`, which only counts kernel-side drops.

**Losses are dated, per-key attribution is not.** `audit_log_lost()` prints `audit_lost=… audit_rate_limit=… audit_backlog_limit=…` at the moment of loss, and every record carries a monotonic serial, so a serial gap brackets the missing events in time. What is missing is which key was lost — "we saw no `net_conn` for that hour" cannot be distinguished from "we saw none because the queue was full".

**Verdict: medium.** This is an integrity issue rather than an evasion, but it is exploitable by anyone who can make noise, and noise is free. Counting `lost` at the time of the *alert* (not on a 5-minute sweep) would make it much harder to exploit; that is not what ships.
