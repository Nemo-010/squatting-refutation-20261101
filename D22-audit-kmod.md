Part of the refutation sweep (#1).

**Claim under test.** `-S init_module,finit_module` records kernel module loads, and `-e 2` makes the ruleset immutable until reboot; together with `kernel.modules_disabled=1` this is the ceiling for in-guest detection (`research/rootkits-that-hide-the-squat.md` §6).

**Kernel WireGuard *is* this rule.** `ip link add wg0 type wireguard` loads `wireguard.ko`, so `finit_module` fires — the corpus's own rule comment says so. The rule catches a WireGuard relay squat, which is the one family interception it actually has.

**The userland class needs no module.** P03 and P05 need no `init_module` and no visible module entry.

**Boot-time load.** A module loaded from an initramfs or by an early systemd unit is in the kernel before `auditd` installs the rule. `-e 2` freezes the ruleset, not the past.

**Reboot with `audit=0`.** `audit=0` on the kernel command line disables audit at boot; `-e 2` does not survive a reboot. An actor with root and a writable bootloader config turns the whole subsystem off and the ruleset never loads. `kexec` achieves the same without a disk change.

**P03 / P04.** The userland class needs no module, and the kernel class can be replaced by an eBPF program attached ahead of the operator's probes — no `init_module`, no visible module list entry.

**`modules_disabled` is one-way, not boot-time.** `Documentation/admin-guide/sysctl/kernel.rst`: it defaults to 0, can be set to 1 at any time, and cannot be set back. Where it is already 1 the LKM class is closed; where it is not, it can be closed now — and it does not touch the eBPF class.

**Verdict: medium, conditional.** On a host with `kernel.modules_disabled=1`, signed modules and a locked bootloader, this closes the LKM half of the kernel class — not the eBPF half, which attaches without loading a module. On a stock cloud image it is a rule that fires on `apt install wireguard` and on very little an attacker does.
