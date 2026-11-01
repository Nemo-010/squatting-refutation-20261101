Part of the refutation sweep (#1).

**Claim under test.** Spamhaus zones with their codes (XBL covers "Free VPN type applications"), SpamCop/CINS/ET compromised lists, AbuseIPDB's score, and Shadowserver's per-vector reports are the external reputation layer (`research/tor-exit-relays-and-IP-reputation.md` §4–§6, `research/open-relays-amplification-reflectors.md` §5).

**Reputation lags the behaviour and the account churns.** A fresh free-tier IP is unlisted by definition; by the time XBL catches a proxyware relay, the operator has rebuilt the host (P19). The corpus's own conclusion — "the business model is account churn" — is also the bypass of every list here.

**XBL is broad on compromised hosts and blind to consenting ones.** Its policy set covers malware, free-VPN proxying, brute-force participation and credential relays — a proxyjacked box is exactly what it is built to catch. What it has no signal for is a host whose *owner* consented: a private seedbox, a CDN relay, a restream. A SOCKS pool is in between — it gets listed only when its victims report.

**AbuseIPDB needs distinct reporters.** The score is a logarithm of distinct reporters with daily decay; a relay with few victims, or one whose victims blame the wrong party, never accumulates a score. The free tier's 1,000 checks/day also caps how much the operator can look.

**Shadowserver covers vectors, not tunnels.** Its reports are per-vector (RDP, open resolver, memcached, SSDP). A CDN tunnel is not a vector and has no report.

**The error code is a trap in both directions.** `127.255.255.0/24` is an error, not a listing; an operator who misreads it gets a false positive, and an operator who has been burned by that stops trusting the feed — which is the state a squatter wants.

**Verdict: medium.** Reputation is the only layer that sees the *effect* rather than the mechanism, and it is the layer the business model is designed to outrun. Read as a lagging indicator it is useful; read as detection it is theatre, which is the corpus's own word for it elsewhere.
