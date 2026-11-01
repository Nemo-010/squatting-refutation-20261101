Part of the refutation sweep (#1).

**Claim under test.** Certificate Transparency (`crt.sh`) shows hostnames pointing at the host, and passive DNS (mnemonic, SecurityTrails) shows who points there — both free and both useful for finding "certificate sprawl" (`research/external-vantage-and-honeypots.md` §2–§3).

**CT sees the name; it does not see the host.** Universal SSL is issued per customer zone, with the customer's domain in the SAN, and Cloudflare logs that issuance to CT by design — so `crt.sh` *does* reveal the squatter's hostname. What is lost is the link from name to machine: the record answers with Cloudflare anycast. A self-signed certificate and a plain-HTTP service are still invisible.

**The relay needs no name.** Outbound-only designs never need an inbound hostname. There is nothing for CT or passive DNS to record: the connection is initiated by the host and the naming is done by the CDN.

**Dynamic DNS and short-lived records.** A free DDNS name that exists for a day is unlikely to be in any passive DNS feed with useful retention, and the feeds the corpus names have coverage biases toward popular domains.

**P22.** If a name is needed, use a subdomain of a legitimate SaaS (a `*.workers.dev`, `*.vercel.app`, `*.ngrok-free.app` name) — CT logs it under the provider, and the association with the host is not visible.

**Verdict: low for squats, medium for exposure.** These tools find a *service* someone published. The corpus's threat is a host being *used*, and the used host has no public name to find.
