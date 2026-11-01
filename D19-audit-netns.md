Part of the refutation sweep (#1).

**Claim under test.** New network namespaces are caught by `-S clone -F a0&0x40000000` plus `-S unshare,setns` (`checks/50-proxyjack.rules`).

**P08 — `clone3` is a different syscall.** The rule matches `clone`, whose flags are the low bits of `a0`. `clone3` takes a pointer to `struct clone_args`, so the flags are not in a register at all and the mask cannot be applied. A runtime that uses `clone3(CLONE_NEWNET)` leaves no `ns_net` event; the rule cannot be fixed by changing the mask, only by adding `-S clone3` with no flag filter (and then every `clone3` fires).

**Create it before the rules load.** `auditd` installs its rules during boot. A namespace created by an initramfs, by a systemd unit ordered before `auditd.service`, or by a container runtime starting at boot is simply older than the ruleset. The squat then enters it once and stays.

**A container runtime is the normal producer.** The corpus's own deployment recommendation is a container. `runc`/`crun` create namespaces through the kernel's clone paths, sometimes `clone3`; the event, if it appears, belongs to the runtime's pid, not the tenant.

**`setns` is the loud one.** The rule matches every `setns` with no filters at all: it fires on `nsenter`, on `ip netns exec`, on every container start on the host, and on the operator's own troubleshooting. That is a rule that trains the operator to ignore it.

**Verdict: medium.** Namespace creation is a real signal, but the rule as written catches one of the two syscalls that create one, and its `setns` companion is pure noise. Detecting the *result* — a new netns inode with no orchestrator object behind it — is the check that would work, and it lives in D08.
