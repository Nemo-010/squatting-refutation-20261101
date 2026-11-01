Part of the refutation sweep (#1).

**Claim under test.** Attribute before acting: RDAP gives the abuse contact for the address, PTR gives a name, ASN/prefix gives context (`research/tor-exit-relays-and-IP-reputation.md` §7, `research/external-vantage-and-honeypots.md` §1.5–§1.6).

**Abuse contacts are missing, and the fallback is not automated.** The corpus measured a RIPE object without `abuse-c` returning nothing (`SE-TORNET`), and the same section names the fallback: `abuse@ripe.net`. A parser that treats an empty RDAP `entities` array as "no abuse desk" is the defect; the contact usually exists one step away.

**Reseller and NAT space.** Wholesale/resold prefixes put the abuse contact several layers from the operator. The contact answers for a range, not for a tenant.

**Attribution is not prevention.** Even a perfect RDAP lookup produces a name and an email. The squatter's account is churnable (P19); the operator's leverage is the abuse desk's queue, which the corpus itself calls slow.

**PTR is provider-settable, not tenant-controlled.** A reverse record can say `mail.example.com`; it is evidence of nothing, and it may be inherited from the previous tenant of the address.

**Verdict: low.** This is due process, not detection, and the corpus presents it correctly as the last step. The one finding worth acting on is the `abuse-c` gap: an operator who assumes the complaint landed is wrong, and the check has no fallback.
