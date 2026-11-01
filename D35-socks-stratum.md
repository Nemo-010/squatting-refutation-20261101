Part of the refutation sweep (#1).

**Claim under test.** A SOCKS5 CONNECT is the 4-byte constant `05 01 00 01`, SOCKS4 `04 01`, and XMRig's first stratum request begins `{"id` (`0x7b226964`) (`research/network-flow-heuristics-proxyjacking.md` §5, `research/ci-runners-and-build-farms.md` §5).

**A SOCKS5 request is a 4-byte constant in one of three cases.** RFC 1928's request is `VER CMD RSV ATYP`, and ATYP is `01` (IPv4), `03` — hostname, the normal case — or `04` (IPv6). A client sending a hostname starts `05 01 00 03`, not `05 01 00 01`. Worse, the first bytes on the wire are the greeting (`05 NMETHODS …`), because method selection precedes the request, so a payload-anchored match on `05 01 00 01` misses the normal case *and* the first flight. SOCKS4's `04 01` has no such split.

**Both constants are optional anyway.** SOCKS5 over TLS (stunnel, `gost -L socks5+tls`, sing-box) puts a TLS record first. HTTP CONNECT — `CONNECT host:443 HTTP/1.1` — is the other universal proxy method and appears nowhere in the corpus's rules. Shadowsocks, VMess and Trojan have no SOCKS phase. Stratum over TLS (`stratum+ssl`) or Stratum V2 with Noise framing removes `{"id`.

**Segmentation defeats the offset arithmetic.** The corpus notes that a segmented login is missed; the same is true of a split SOCKS greeting, which a client may send in two writes (`05 01 00` then `01`).

**P14, not P06.** The constant appears on the socket the *client* uses. A relay that terminates the client's SOCKS locally and forwards over its own protocol shows the operator only its own protocol.

**Verdict: medium.** These are exact signatures for the default configuration of two tools. Both tools ship TLS wrappers, and the corpus's own §5 recommendation ("Zeek ships a SOCKS analyzer … DPD-driven") is the stronger version of the same check — but the *relay* the corpus is worried about is the one that terminates SOCKS locally and never puts it on the wire.
