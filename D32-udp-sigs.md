Part of the refutation sweep (#1).

**Claim under test.** Missing SNI is a proxy shape; QUIC Initials start `0xC0`–`0xCF` and a `0xC0` stream with no `quic.log` entry is "something else wearing 443"; WireGuard handshake *initiations* are fixed 148-byte packets starting `01 00 00 00` (response 92 B, cookie reply 64 B); STUN is identified by the `0x2112A442` cookie (`research/network-flow-heuristics-proxyjacking.md` §2–§3).

**P16 — send the SNI.** REALITY's whole design is a valid SNI for a real site. "SNI absent, or ALPN not h2/h1" describes a client that did not bother, not one that read the paper.

**QUIC: the default already mimics HTTP/3.** Hysteria's documentation says the protocol "by default … mimics HTTP/3", so `quic.log` gets a version, an SNI and an ALPN. The `0xC0` first-byte test is satisfied by every QUIC endpoint including real browsers. The anomaly rule needs the *absence* of a parse, which a well-formed h3 tunnel does not produce.

**Salamander makes it random, which the rule treats as unknown.** With obfuscation enabled, hysteria "scrambles every packet into seemingly random bytes"; the first byte is no longer `0xC0` and no QUIC parser will log it. That is a *different* anomaly (unknown UDP on 443) and one that fires on every unparsed UDP flow, including games and VoIP. Gecko adds random fragmentation, which removes the size regularity too.

**AmneziaWG deletes the 148-byte constant.** Its obfuscation parameters add a junk prefix and shift sizes specifically so the fixed handshake stops being fixed; the filter finds vanilla WireGuard initiations and nothing else.

**Port hopping moves the flow off 443.** hysteria2 ships port hopping: the server listens on a range (`listen: :20000-50000`) and the client may send to any port in it. The anomaly rule is keyed to udp/443, so a hopped tunnel either never fires it or joins the unparsed-UDP bucket with games and VoIP.

**STUN is not a detector.** The cookie appears in WebRTC, every video call, most games and a large share of mobile apps. A rule that fires on it is a rule the operator disables in week one.

**Verdict: medium.** Each signature is precise and each has a published, widely deployed option that erases it. The class as a whole is also blind on a switched network without a mirror and to anything macvlan/SR-IOV (P11) sends — the capture has to see the packet first.
