Part of the refutation sweep (#1).

**Claim under test.** Established-socket inspection finds the outbound-only families: `ss -tnp state established '( dport = :7844 or sport = :7844 )'` for cloudflared, long-lived sessions to 443 with a non-browser name, and "a process that connects but never accepts" (`checks/triage.sh`, `REPORT.md` §4).

**The default tunnel is not on that port at all.** cloudflared's `--protocol` defaults to `auto`, which its docs describe as configuring QUIC when UDP works — QUIC is UDP 7844. The check is `ss -tnp`, TCP. A stock `auto` tunnel produces no matching row; only `--protocol http2` does. The filter needs a UDP twin (`ss -unp '( dport = :7844 )'`), which the corpus does not ship.

**P15 — change relay.** 7844 is one vendor's edge port. ngrok, Pinggy, localtunnel, bore, zrok, Pangolin, Tailscale DERP and plain `ssh -R` publish over 443 or 22, and none appears in the filter.

**QUIC is invisible to `-t`.** hysteria2, TUIC, MASQUE/WARP and every QUIC-based proxy run over UDP/443 and never appear in the TCP established list. The corpus's "UDP sockets" dump is unfiltered; there is no established-QUIC equivalent of the 7844 check.

**P16 — the destination is legitimate by construction.** CDN-fronted relays terminate on a first-party CDN address with a valid certificate. `grep -vE 'chrome|firefox|apt|curl'` filters by *process name*, which P13 sets to `nginx` or `systemd-resolve`.

**P10.** Another netns. Host `ss` shows nothing, and the veth pair carries bytes with no attributable socket.

**P07.** A socket opened at boot and inherited has no `connect()` in the window the operator is watching, and the "connect but never accept" ratio was never measured for it.

**P17.** Reconnect on a timer: each session stays under whatever duration threshold defines "long-lived", and the tunnel is still up 99 % of the time.

**Verdict: medium.** It catches `cloudflared --protocol http2` — a real case, but not the default one — and almost nothing else. The deeper problem is that "one encrypted flow to a reputable CDN on 443" is simultaneously the detection target and the description of every HTTPS client; the corpus concedes this in §8 and then still lists duration and destination as signals.
