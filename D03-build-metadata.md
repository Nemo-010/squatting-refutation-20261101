Part of the refutation sweep (#1).

**Claim under test.** Renaming defeats name matching, but Go and Rust binaries keep build metadata: `go version -m /proc/<pid>/exe` recovers module paths and `strings` finds upstream URLs (`research/proxy-tunnel-host-artifacts.md` §1.3).

**Choose a language that leaves nothing.** The whole family of self-service relays has C implementations: `microsocks`, `dante`, `3proxy`, `tinyproxy`, `socat`. None carries a module graph. A stripped C binary exposes only its own literals.

**Rewrite the module path, do not fight the tooling.** `go mod edit -module github.com/something/boring` before the build changes the buildinfo and the `github.com/...` literal grep with no code change — cheaper than `garble` or section scrubbing, both of which do work. `strip` on a Rust binary removes symbols but not string literals — so the string check still has a target, which is why the actor replaces the literals rather than the symbols.

**P02.** `dlopen` a proxy core into an interpreter or an existing daemon and there is no separate binary to inspect at all.

**The check needs a toolchain.** `go version -m` requires Go installed on the defender's host. The corpus's own fallback is `strings`, which requires binutils — so on a minimal server image neither half runs, and the check has no third option.

**Verdict: medium.** Good as a triage lead on a *named* Go binary whose provenance is already in question. It cannot establish that a binary is a relay, and it fails outright to a C implementation — which is the cheaper build for an actor who reads this document. It also cannot distinguish `cloudflared` installed for a legitimate tunnel from `cloudflared` installed for a squat: same module path, same version string, same bytes. That case is `indistinguishable` and only the destination/purpose check separates them.
