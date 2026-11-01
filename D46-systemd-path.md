Part of the refutation sweep (#1).

**Claim under test.** `systemd.path` with `PathExists=` / `DirectoryNotEmpty=` needs no packages, and the traps are `TriggerLimitIntervalSec` / `TriggerLimitBurst` (defaults 2 s / 200, systemd 250): after the burst limit the unit fails and silently stops watching (`REPORT.md` §4, `research/external-vantage-and-honeypots.md` §6.2).

**The trap is the bypass.** The corpus documents it correctly and still recommends the feature. From `systemd.path(7)`: "If the limit is hit, the unit is placed into a failure mode, and will not watch the paths anymore until restarted." Two hundred changes in two seconds — trivial to produce on any file the path unit watches — and the tripwire is off. The attacker does not need to know that a path unit exists; they need to touch the watched path noisily once.

**Dotfiles are ignored.** Same man page: "Note that files whose name starts with a dot (i.e. hidden files) are generally ignored when monitoring these paths." A canary named `.credentials` in a watched directory is not seen. The implementation agrees: `DirectoryNotEmpty=` evaluates `dir_is_empty(path, /* ignore_hidden_or_backup= */ true)`, so a directory containing only hidden files still reads as empty and does not trigger.

**Checked against the current documentation, not the corpus's summary (2026-11-01).** `systemd.path(7)` for systemd 262 has both defaults verbatim — "defaults to 2s" and "defaults to 200", "Added in version 250" — and adds that the trigger limit "is enforced before the service activation is enqueued", so the failure happens even when the activated service itself would have refused to start. "Set either to 0 to disable any form of trigger rate limiting" is the only recommended configuration, and it is not what the corpus's example shows.

**`PathExists=` fires on a transition.** A file that is created before the unit starts and never removed produces one activation at most. A canary that is *read* (the interesting event) never changes existence at all.

**The corpus documents the disarm and the mitigation.** It says silence looks like safety and recommends polling `systemctl is-failed` and adding `OnFailure=`. The defect is that a bare `systemd.path` is recommended before either is wired — and a unit that is already in `failed` state is one line in a status page nobody reads.

**Verdict: high.** This is a documented, one-command, self-blinding tripwire recommended as a control. Either the limits are set to 0 (no rate limit) and the failure mode is monitored, or the feature is a liability.
