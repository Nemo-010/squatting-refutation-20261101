Part of the refutation sweep (#1).

**Claim under test.** `internetdb.shodan.io/<ip>` gives ports, hostnames, tags and vulns with no key, and Censys Platform API is the keyed equivalent; diff outside against inside (`research/external-vantage-and-honeypots.md` §1.1–§1.4).

**The tunnel family has no listening port.** InternetDB is a port index. `cloudflared`, ngrok, `ssh -R`, tailscaled and the CDN relays publish nothing to scan; the outside view is an empty record, which reads as "clean".

**Scan freshness is not a promise.** InternetDB reflects Shodan's last observation, which may be months old on a host nobody targets, and Shodan's coverage is its scanner's schedule, not the whole port space. A listener that exists only while the tenant is awake is missed by any schedule.

**The diff is against the wrong thing.** "Outside minus inside" finds a port the host thinks is closed; the squat's interesting direction is *outbound*, where the outside view has nothing to say.

**CDN fronting moves the answer to the CDN.** A service published through Cloudflare resolves to Cloudflare addresses; scanning them tells you about Cloudflare.

**Verdict: low for the threat, medium for a different one.** It is an excellent free check for "is my Jupyter/RDP/Docker exposed", which the corpus also covers. Presenting it in the squatting detection tier conflates "exposed service" with "host being used as a relay", and the relay case is invisible to it by construction.
