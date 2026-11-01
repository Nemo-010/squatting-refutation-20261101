Part of the refutation sweep (#1).

**Claim under test.** RDP/VNC/xrdp are found by listener and port checks, `/proc/net/tcp` hex, `xrdp-sesman` auth-failure counts, `qwinsta`, Security event IDs 4625/4624 type 10/1149/21/22/25, service-name regexes, and connection-count heuristics (`research/windows-rdp-vnc-and-remote-desktop-abuse.md` §6).

**Use a remote-access tool that is not in the list.** AnyDesk, RustDesk, ScreenConnect, Chrome Remote Desktop, TeamViewer, MeshCentral and Guacamole all dial out over 443. No 3389, no `rdp-tcp#N`, no `TermService` event, no listener. The corpus lists these tools under "shadowing" and then detects them by *service name*, which is a string in the registry.

**Reverse-tunnel the desktop.** xrdp or VNC bound to `127.0.0.1` and published with `ssh -R` or `cloudflared` shows one loopback listener and one outbound flow — the exact shape the corpus calls normal for an application.

**The logs are the attacker's to edit.** The corpus says actors "manipulate or clear the security event logs" and then recommends reading 1149 and 21/22/25 *when Security looks clean*. That is a good instinct and a bad control: an actor with admin clears all four sources, and the check has no independent copy.

**NLA moves brute force to type 3.** The corpus notes it: under NLA, failures can land as logon type 3, so a type-10 filter misses them. The fix is to count types 3 and 10, which then includes every failed network logon on the host — a different, much noisier detector.

**Valid credentials produce no failure events at all.** The successful logon is type 10, which is what every legitimate RDP session is.

**Verdict: medium.** The listener and config checks are exact and worth running. The behavioural half is a log-hunting exercise against an adversary who owns the log, and the tool-substitution bypass (AnyDesk et al.) is one download.
