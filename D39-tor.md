Part of the refutation sweep (#1).

**Claim under test.** `check.torproject.org/api/ip`, `torbulkexitlist` (1,225 IPv4 lines measured), and Onionoo `?search=<ip>` give a free, authoritative "am I an exit" answer, with `exit_addresses` scoped to the last 24 h (`research/tor-exit-relays-and-IP-reputation.md` §3).

**The list is IPv4-only.** The corpus measured **0 of 1,225** lines containing `:`; re-fetched on 2026-11-01 it is **1,228 lines, still 0 with a colon** — three entries of churn in a day. An IPv6-only exit does not appear, and `grep -Fx "$MYIP"` on an IPv6 address matches nothing. A dual-stack host that publishes only AAAA is unlisted.

**Churn does not beat it — the list and the usability come from the same consensus.** A relay gets `Exit`/`Fast` from the bandwidth authorities and clients learn of it from the consensus; `torbulkexitlist` and Onionoo are built from the same network status. The unlisted window is bounded by one consensus/Onionoo cycle (minutes to an hour), and during that window the exit is not usable as an exit either. So churn costs the squatter a relay, not the checker a sighting. The real gap is *non-Tor* relays.

**Do not be an exit.** The threat model is not only Tor. A SOCKS5 relay sold to a proxy pool (the actual proxyjacking business) is never a Tor relay and never appears in any of these sources. The Tor check answers a question the corpus asks because it is free, not because it is the threat.

**NAT and shared addresses.** Provider NAT means the observed egress IP is shared; a listing implicates every tenant, and a clean result proves nothing about any one of them.

**Verdict: medium.** It is authoritative and free, and for *Tor exits* it is close to complete on IPv4. As a general "is this host relaying" check it covers one niche, and the niche is the one with the loudest complaint channel — which is exactly the niche a professional would avoid.
