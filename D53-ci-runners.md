Part of the refutation sweep (#1).

**Claim under test.** A runner is found by `Runner.Listener`/`Runner.Worker`, the `actions.runner.*` unit, the inherited `GITHUB_ACTIONS=true`, per-runner `IPAccounting`, `ss cgroup`, GitHub's `meta` IP ranges, the job hooks, two static workflow checks, and Harden-Runner (`research/ci-runners-and-build-farms.md` §5–§7).

**P23 — unset it.** `GITHUB_ACTIONS=true` is in the environment of everything a job spawns, and `env -u GITHUB_ACTIONS` removes it from the payload's environment. The parent chain still reaches `Runner.Worker`, which is the corpus's other half of that check — so run the payload detached (`setsid`, `systemd-run --user`, P12) and both halves are clean.

**Rename the agent.** `Runner.Listener` is a .NET assembly; copy it and the process name changes. The `actions.runner.*` unit name changes with the service name. Neither is authenticated.

**P12 — leave the runner's cgroup.** `systemd-run --scope` from the job user, or a rootless container, puts the payload where `ss cgroup …actions.runner…` does not match and `IPEgressBytes` does not count.

**It is the wrong CI.** `gitlab-runner`, Jenkins agents, Buildkite, Drone, Woodpecker — all free-compute agents on a borrowed host, none with the names or the env var.

**Self-hosted runners are not in `meta.actions`.** The IP-range check answers "did this come from GitHub-hosted infrastructure". A squatter running their own runner on your host sends from *your* IP, which is not in the list, and the check reports "not github-actions" — which reads as exculpatory.

**The hooks snapshot boundaries.** They capture sockets at job start and job end; anything that opens and closes inside the job is invisible, and P17 makes every flow short. Harden-Runner has a documented `disable-telemetry` option and only covers public repos.

**Verdict: medium-high for hosted-runner abuse, low for a squat on your own host.** The corpus is really describing GitHub's problem. Applied to a single server, every check names a tool the attacker chose, and the choice is free.
