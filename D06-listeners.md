Part of the refutation sweep (#1).

**Claim under test.** A listener inventory plus a port table finds the family: `ss -tulpn`, flag anything on 19/111/123/389/1900/3702/5353/11211, non-standard ports, and "unusual port + Go binary + no package owner" (`checks/triage.sh`, `research/proxy-tunnel-host-artifacts.md` §2.2).

**P15 / P14 — the fastest-growing family listens on nothing.** `cloudflared`, ngrok, frpc, rathole, chisel-client, `ssh -R`, tailscaled and the CDN-published relays all dial out. There is no row for them in any listener table, and the corpus says so — which makes the table a detector for the *slow* families only.

**Camouflage the port.** Bind 443 and speak TLS. On a host that already serves HTTPS, one more 443 listener is an ordinary web server; with REALITY or a valid certificate for a throwaway name it also answers probes correctly. The port table's "443 tcp — xray / sing-box / naive" entry is the same row as "443 tcp — nginx".

**A loopback listener behind a real front end.** nginx/Caddy already listen on 80/443. With the `stream` module, `stream { server { listen 443; proxy_pass 127.0.0.1:1080; } }` forwards raw bytes to a SOCKS5 listener, so the public socket is a legitimate front end and the actual proxy is loopback-only. (`http`-module `proxy_pass` cannot do this: it emits an HTTP request, and SOCKS5 is not HTTP.) `ss -tulpn` still shows the loopback socket, but on a host that has applications it is an application socket — the false-positive class the corpus itself warns about.

**P07 — the event, not the attribution.** systemd socket activation creates the listener in PID 1 and passes a copy of the fd to the service, so there is no `socket()` or `bind()` in the payload's history. `ss -p` will list both holders, so this hides the creation event, not the owner; `systemctl list-sockets` and `ss --cgroup` are the unit-level views, and neither is in the check.

**P10 / P25.** A container's listener is in another netns and absent from host `ss`. A rootless podman container needs no Docker CLI, so the `docker ps` half of the check finds nothing either.

**No listening socket at all.** A userland TCP stack over `AF_INET`/`SOCK_RAW` answers SYNs from a raw socket, so there is no `bind` and no listener row. It needs `CAP_NET_RAW` to send *and* root/`CAP_NET_ADMIN` to install the rule that suppresses the kernel's own RST for the claimed port — the kernel answers an unbound port itself, so `CAP_NET_RAW` alone is not enough.

**Verdict: high.** Port tables are a lookup aid for a human who already suspects a host, not a detector. Every entry in the table is a configurable default, and the corpus's own caveat — "xray/sing-box/tuic/naive have no compiled-in default listener" — means the signature degenerates to "unusual port", which is the weakest possible test on a box running anything custom.
