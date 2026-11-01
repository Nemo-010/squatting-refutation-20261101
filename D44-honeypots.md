Part of the refutation sweep (#1).

**Claim under test.** `endlessh` on 2222 (10 s delays, `MaxLineLength 32`) and Cowrie (full SSH/Telnet emulation, JSON events) catch the entry rather than the workload, for almost no cost (`research/external-vantage-and-honeypots.md` §5).

**They catch the vector they emulate, and only from a scanner.** Proxyware installs on a host that is already compromised: leaked credentials, a vulnerable web app, an exposed notebook, the Docker API. None of those paths touches SSH. The corpus does describe non-SSH decoys elsewhere — a web bug in a wiki page, a DNS token in a fake `.env`, tokenised credentials — but those detect the intruder's *reconnaissance reads*, not the squat.

**A scanner that fingerprints skips.** Cowrie's emulation is good and its banner is not the host's; `endlessh` holds a connection open with no banner at all for 10 seconds, which is itself distinctive. An actor who scans 22, sees a tarpit, and moves on has cost the operator nothing and yielded only the scanner's IP.

**Port displacement.** `endlessh` on 2222 is not 22. The scan traffic the corpus wants to capture goes to the real SSH port (22) where the operator cannot tarpit it without breaking their own access.

**Nothing logs the squat itself.** Even a perfect Cowrie capture gives an IP and a password; the payload ran elsewhere. As a *detector* of squats it is a source of leads about attempts, which is a different graph.

**Verdict: low.** These belong in an entry-detection programme. Listed under "cheapest, underused" in a document about squatting, they set an expectation that the source of the squat is scanning, which the corpus's own case studies contradict — every documented chain in §2 starts from an exposed service, not from a brute-forced SSH.
