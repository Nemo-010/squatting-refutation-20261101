Part of the refutation sweep (#1).

**Claim under test.** MSE/PE leaves a shape: each side's first flight is 96–608 bytes of uniform random bytes with no recognisable prefix, on a port that is neither 80 nor 443, followed by sustained bidirectional transfer. Detection is flow statistics, not `content:` (`research/torrent-seedbox-detection.md` §2.6).

**The port clause is the bypass.** "A port that is neither 80 nor 443" — run the seeder on 443. The clause accepts everything else, including 8443 and 8080, so those are not evasions at all; the exclusion is a single hole and the attacker picks it.

**Random is not distinguishable from other random.** 96–608 bytes of high-entropy data with no prefix is what obfs4, Shadowsocks and every delivered-TLS wrapper looks like. A real TLS ClientHello does *not* match: it begins `16 03 01`, exactly the recognisable prefix the rule excludes. The remaining shape is "small first flight, then sustained bidirectional transfer", which is also a WebSocket, a database connection and a video call.

**P11 / P15.** Put the seeder behind the CDN tunnel and the operator sees one TLS flow to a CDN — the shape is the tunnel's, not the client's.

**P17.** Pace the upload. The sustained-transfer clause is satisfied by any rate above nothing; below the operator's threshold it is just a connection.

**Verdict: medium, and the file says so.** This is the most honest section of the corpus: it states that everything above is worth zero against `required` and offers shape instead. Shape is a weaker detector than a constant by an order of magnitude, and the clause that would make it specific (non-443) is the easiest thing to change.
