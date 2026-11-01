Part of the refutation sweep (#1).

**Claim under test.** Meter at spawn (`IPAccounting=yes` via a systemd scope or `systemdspawner`), police with `socket cgroupv2 level 2 "<path>" limit rate over 2 mbytes/second drop`, allow-list with `IPAddressDeny=any` + `IPAddressAllow=localhost`, and stop inbound with `SocketBindDeny` or an input-hook drop (`research/jupyter-multitenant-detection.md` §3–§4).

**P21 — the allow-list hands over DNS.** A notebook that may only reach `localhost` plus `10.0.0.53/32` can still run `dnstt`/`iodine` through that resolver: the resolver recurses to the tunnel's authoritative server, which is the egress. The policy is satisfied exactly; no denied address is ever contacted. The corpus anticipates this ("add `IPAddressAllow=10.0.0.53/32` if you keep a resolver") without noticing that the resolver is a general-purpose channel.

**P12 — the cgroup is not the boundary the tenant respects.** `systemd --user` and rootless podman put the payload in a scope under `user.slice`, not under `jupyter-alice-singleuser.service`. `socket cgroupv2 level 2 "<that path>"` does not match, accounting does not see it, and the notebook can start either without any privilege beyond the shell it already has.

**Sockets do not migrate.** The corpus states the rule and the consequence ("apply at spawn, never retrofit"). The same fact makes the control fragile: a session that starts before the policy, or in a sibling scope, is outside it for its whole life.

**`SocketBindDeny=any` is unusable as documented.** The kernel must bind five loopback ZMQ ports, and the grammar has no address selector. The corpus recommends the input-hook drop instead — which needs the exact cgroup path, breaks under `DynamicUser` and unit restarts, and needs `NFTSet=` (systemd 260) to track cgroup IDs.

**Verdict: medium-high.** `IPAddressAllow` with no resolver allowed is the one control in the corpus that removes the capability rather than describing it. The version that ships includes the resolver, and once DNS is allowed, the allow-list is a bandwidth cap on the tunnel, not a block.
