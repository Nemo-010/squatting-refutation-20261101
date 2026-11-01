Part of the refutation sweep (#1).

**Claim under test.** Canarytokens fire on any access or resolution; `chattr +i` is a write tripwire; `atime` under `relatime` moves once and can be read as a signal (`REPORT.md` §4 tier 6).

**`chattr +i` protects nothing against the privilege that reads it.** It is cleared by root or `CAP_LINUX_IMMUTABLE` — exactly the capability the installer has, and the corpus's own Kinsing description has the deploy script "removing the immutable file flag" first. The marker is a tripwire against *accidental* deletion, not against an adversary.

**It is write-only.** The corpus says so: the file "can not be opened in write mode" and reads are not detected. A credential canary that is read is the case the operator cares about, and `chattr +i` is blind to it.

**`atime` is defeated by the mount and by repetition.** `noatime` (common on container hosts and tuned filesystems) never updates; `relatime` updates only when the previous access is more than a day old, so the first read moves it and the next thousand do not.

**Block the token domain — at the resolver, not in `/etc/hosts`.** `/etc/hosts` is an exact-match table with no wildcard support, so a `*.canarytokens.com` line matches nothing. A wildcard-capable resolver (dnsmasq `address=/canarytokens.com/0.0.0.0`, unbound, an RPZ) or a DNS egress allow-list does block it. This only defeats the DNS and web-bug token types: AWS-key, fake-file, document and Windows-folder tokens fire through provider telemetry and never touch a canarytokens domain.

**Verdict: medium.** Canarytokens are the cheapest tripwire that exists and they catch an explorer. They do not catch a payload that was delivered by the operator's own provisioning and never reads a decoy. The `chattr +i` recommendation specifically should be demoted: it is a deletion guard, and the corpus presents it as a detection.
