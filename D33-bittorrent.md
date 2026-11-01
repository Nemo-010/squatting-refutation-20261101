Part of the refutation sweep (#1).

**Claim under test.** The peer handshake is `0x13 "BitTorrent protocol"` at any port; DHT KRPC starts `d1:ad2:id20:` / `d1:rd2:id20:`; the UDP tracker magic is `0x0000041727101980`; `announce_peer` exposes the info_hash in cleartext (`research/torrent-seedbox-detection.md` §2).

**P16 — MSE/PE `required` hides the peer handshake, not the protocol.** Set it and the `0x13426974` handshake, PEX and LSD disappear. DHT is separate plaintext UDP (BEP 5) and tracker announces are a separate protocol (HTTP or the BEP 15 UDP magic), so two of the four constants survive — and for a client that has DHT off, the DHT constants were never there. The corpus states the encryption caveat correctly and over-states its reach.

**Turn DHT off.** A private-tracker client must not use DHT, PEX or LSD (BEP 27). The largest ratio-farming population therefore emits none of the DHT constants by design. The corpus knows this and uses the *absence* as its private-tracker signal, which means the wire signature covers the public-swarm case only.

**HTTPS trackers.** BEP 3 peers announce over HTTP; modern trackers are HTTPS. No UDP magic, no cleartext announce, no bencoded query on the wire.

**Port 443 and a relay.** A seedbox behind a CDN tunnel or a SOCKS5 relay presents the operator with one TLS flow; the BT handshake is inside it.

**Segmentation.** The `tcp[((tcp[12]&0xf0)>>2):4]` match assumes the handshake starts at payload offset 0 of the first segment. A first write split inside the first four bytes defeats it — tcpdump reassembles nothing. Neither TFO (the data rides the SYN and the offset nibble still resolves) nor MSS changes the offset.

**Verdict: medium.** The DHT constants are the cheapest and most reliable packet signature in the corpus, and they only exist for clients that use DHT. The population the file is actually about — private-tracker ratio farming — is defined by not using it.
