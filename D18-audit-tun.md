Part of the refutation sweep (#1).

**Claim under test.** `ioctl(fd, TUNSETIFF, &ifr)` is the highest-value audit rule for tunnel squatting, matched as `-F a1=0x400454ca`, because a TUN device needs `CAP_NET_ADMIN` and cannot be created any other way (`research/auditd-proc-detection.md` §3).

**P14 — do not create a TUN.** A SOCKS5/HTTP-CONNECT relay, `ssh -D`, or `tailscaled --tun=userspace-networking` needs no interface. The rule watches the one thing these designs deliberately avoid. Tailscale's own docs describe userspace networking as the mode for hosts without `/dev/net/tun`, which is exactly the container case.

**WireGuard never calls it.** The corpus notes this and then keeps the rule as "the highest-value rule": `wg`-family interfaces are created over rtnetlink (`RTM_NEWLINK`), not `TUNSETIFF`. A WireGuard or AmneziaWG tunnel is invisible to this rule.

**Netlink gets you interfaces without ioctl.** `veth`, `ipip`, `sit`, `gretap`, `macvlan` and `dummy` are all created through `RTM_NEWLINK`. A relay bridging into a netns can be built entirely from netlink-created devices.

**P06 — the attached fd, not a fresh open.** An fd from `open("/dev/net/tun")` is unattached (`tun_chr_open()` leaves `tfile->tun` NULL); attaching to an existing persistent interface still requires `TUNSETIFF`, so this fires. The variant that works is an interface attached earlier by a legitimate process, with the already-attached fd passed in by `SCM_RIGHTS`.

**b32 is not matched.** `tun.c` provides `tun_chr_compat_ioctl`, and the rule is `arch=b64` only. A 32-bit process calling `TUNSETIFF` — the same request word — matches nothing, the same hole D17 raises for `socket`/`connect`.

**Verdict: medium.** It catches OpenVPN, `wireguard-go`'s TUN path and a hand-rolled `ip tuntap add`. It does not catch the two designs the corpus recommends elsewhere (userspace proxy, CDN relay) or any WireGuard interface created over netlink. The rule is worth keeping; the "highest-value" label is not supportable.
