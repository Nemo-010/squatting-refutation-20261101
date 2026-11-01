Part of the refutation sweep (#1).

**Claim under test.** RITA does beaconing, long-connection and DNS-tunnelling detection out of the box with severity buckets `base 50 / low 70 / medium 90 / high 100`; `conn.log` gives `duration > 3600` and bytes > 1e5 as a relay shape; ntopng raises REMOTE TO REMOTE for a host talking outside its subnet (`research/network-flow-heuristics-proxyjacking.md` §8, §9).

**Beaconing needs a beacon.** A relay's traffic is driven by its clients — it is bursty, not periodic. A *seedbox* is a constant upload; a restream is a metronome. The detector is tuned for C2, and the workload it is being pointed at does not beacon.

**Jitter defeats the score.** Where the payload does poll (a subscription refresh, a keepalive), randomising the interval is P17 and drops the score below 50.

**The long-connection query is a false-positive generator.** SSH sessions, database pools, WebSockets, video, and every legitimate monitored connection satisfy `duration > 3600 && bytes > 1e5`. On a normal host the output is a page of results the operator learns to skim. P17 — reconnect every 50 minutes — puts the tunnel under the threshold anyway.

**REMOTE TO REMOTE does not fire.** It triggers when a device talks directly to another device outside its subnet. A CDN-fronted relay's peer is the CDN, which is normal traffic for every host. A proxyjacking relay's peer is its own broker — one remote, not remote-to-remote.

**Verdict: medium.** RITA is a good C2 tool. Proxyjacking is not C2, and the corpus is applying a beacon detector to a workload whose whole point is to look like ordinary client traffic. The one part that survives — long connections with steady bytes — is exactly what a legitimate long connection looks like.
