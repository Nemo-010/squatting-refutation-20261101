Part of the refutation sweep (#1).

**Claim under test.** vnstat keeps months of per-interface history without capturing, and `vnstat --alert output exit type d condition above limit 25 GiB` is the cron-friendly canary (`research/per-user-network-accounting.md` §4).

**P11 — a different interface than the one being counted.** macvlan/ipvlan give the tenant a device that transmits with its own MAC; the host's `eth0` counters do not see those frames, and `vnstat -i eth0` reports a quiet host. An SR-IOV VF or a passthrough NIC does the same one level down. Both are ordinary cloud/container networking features.

**It is one interface, chosen by hand.** Hosts have `docker0`, `br-*`, `bond0`, VLANs, `veth*` and `tun0`. vnstat is configured per interface; a tunnel over `tun0` with a rule watching `eth0` is invisible to the alert, and the default config does not enumerate what the host actually has.

**The shipped command does not parse.** `--alert` takes ordered, typed arguments (mode, exit status, period, type, limit); bare words like `output exit … condition above` are rejected, and a malformed `--alert` prints its own help and exits. The working shape is `vnstat -i eth0 --alert 1 5 d tx 25 GiB`, with an exit status chosen so a real vnstat error (always 1) is distinguishable from the alert. As quoted, the canary is a cron job that reports nothing.

**P19.** 25 GiB/day is ~2.4 Mbit/s average. That is a working relay, a working seedbox on a private tracker, and well inside every free tier's published allowance.

**P17.** Duty-cycle: ~10 Mbit/s for 6 hours, or full rate for ~35 minutes, stays under 25 GiB/day. The alert never fires and the relay runs while the operator is asleep.

**P24.** The counters are kernel-side and hard to lie to — the check's real strength — but the *sampling and thresholding* around them are userland and configurable, and `vnstat`'s daemon can be stopped.

**Verdict: medium.** It is the most rootkit-resistant *on-host* byte source in the corpus — not the most resistant overall; that is the provider's bill — and the right thing to baseline. It answers "is this host sending more than usual" and cannot answer "is this host a relay", and every threshold is a number the squatter can read.
