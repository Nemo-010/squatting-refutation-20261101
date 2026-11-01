Part of the refutation sweep (#1).

**Claim under test.** A Jupyter kernel channel is a WebSocket: HTTP `101 Switching Protocols` on a non-443 port followed by a long-lived, low-rate, bidirectional flow is the notebook shape. And `capture_loss.log` `percent_lost` is the only honest answer to "am I seeing this traffic" (`research/network-flow-heuristics-proxyjacking.md` §6, §9).

**Jupyter is deployed behind a reverse proxy.** The standard production layout is nginx/Caddy terminating 443 and proxying to `127.0.0.1:8888`. The 101 now happens on 443 and the "non-443 port" clause — the only thing that made the shape specific — is gone. JupyterHub's own default is 8000, also proxied.

**WebSocket is everywhere.** Every chat app, dashboard, IDE and terminal-over-web product uses the same 101 + long-lived flow. The shape does not identify a notebook; it identifies the last decade of web applications.

**It measures ACK gaps in what Zeek saw, and nothing else.** `percent_lost` is the share of ACKs whose data was not seen, so a *local* capture drop raises it — the number is not fooled by ring-buffer loss. What it cannot report is traffic that never reached the sensor: a tap that was never installed, a mirror that misses a VLAN, a macvlan/SR-IOV tenant (P11). And it is ACK-derived, so UDP/QUIC contributes nothing at all — a 0 % reading says nothing about the hysteria2 tunnel D32 covers.

**P11.** A macvlan/SR-IOV tenant's traffic never crosses the mirrored interface. The sensor reports 0% loss because it lost nothing — it was never in the path.

**Verdict: medium for the WS shape (a notebook detector, not a squat detector), low for the integrity claim.** `percent_lost` measures the sensor's own ACK-visible gaps, not the network's coverage, and not UDP at all. The corpus treats a clean number as licence for a negative; the number cannot carry that.
