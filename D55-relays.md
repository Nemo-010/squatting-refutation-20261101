Part of the refutation sweep (#1).

**Claim under test.** Open relays and reflectors are found by listener ports (19, 111, 123, 389, 1900, 3702, 5353, 11211), a TX/RX packet-size ratio, `/proc/net/snmp` UDP counters, conntrack entry rates, an SMTP relay self-test, outbound 25/587/465 sessions, and Shadowserver's reports (`research/open-relays-amplification-reflectors.md` §5).

**P22 — no SMTP socket at all.** A compromised host sends mail through a SaaS API (SendGrid, SES, Mailgun) or a bot API (Telegram, Discord) over 443. There is no port 25 session, no queue, no `relay=`, and nothing for the self-test to probe. This is how most bulk mail has worked for a decade, and the corpus's SMTP section does not mention it.

**Rate-limit the reflector below the ratio.** A DNS resolver that answers a stranger with a small response has a TX/RX ratio near 1. The amplification factor is what makes a reflector useful, and it is also what makes it detectable; an operator who caps it is neither.

**Whitelist the resolver and keep it useful.** `allow-recursion { trusted; }` makes the open-resolver test (a probe from outside) return REFUSED, so the check reports "closed". The resolver still recurses for anyone on the trusted list — including a tenant.

**Reflect with someone else's infrastructure.** Rent the amplification from a provider that offers it, or use a public resolver. The victim's traffic never touches the squatted host.

**conntrack timeouts hide the burst.** `nf_conntrack_udp_timeout` is 30 s; a low-rate reflector produces a flat entry count.

**Verdict: medium.** The listener check is exact for the default configs and worth keeping. The behavioural checks assume the relay is maximising amplification, which is the one thing a careful operator does not do.
