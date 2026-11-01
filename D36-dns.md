Part of the refutation sweep (#1).

**Claim under test.** DNS tunnelling shows up as leftmost-label entropy above ~3.6, TXT/NULL query volume, and per-domain query rate; the threshold is a starting point, not a published constant (`research/network-flow-heuristics-proxyjacking.md` §7).

**The threshold is the evasion.** The corpus gives its own reference points: `google` → 1.92, `xqztvkfjrnwpl` → 3.70, a hex string → 4.62. A tunnel that wants to stay under 3.6 encodes in short dictionary-like labels — a wordlist plus a counter — and pays for it with more queries. The detector's author chose the number; the attacker optimises against it.

**DoH/DoT leaves the DNS path entirely.** A tunnel wrapped in DNS-over-HTTPS is a TLS session to a resolver on 443. Zeek's `dns.log` sees nothing, `quic.log` sees an h2/h3 session, and `conn.log` sees a long-lived flow to a well-known resolver — which is indistinguishable from every browser on the host (P16, P22).

**TXT/NULL volume is a shape, not a constant.** Split the payload across A records, use short TTLs, keep the query rate under any per-domain threshold. `iodine` and `dnstt` have knobs for exactly this.

**P24.** A resolver that answers a high query rate is the same resolver that serves the host's legitimate lookups; flooding it makes the *legitimate* queries anomalous, which is how the operator's own baseline is destroyed.

**Verdict: medium.** Entropy is a real signal for a naive tunnel. It is not a control: the number is public, the encoding is the attacker's choice, and the modern deployment (DoH) removes the observable from the log the check reads. The corpus is honest that DNS is its thinnest slice; this is why.
