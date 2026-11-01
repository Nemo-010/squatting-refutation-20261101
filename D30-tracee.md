Part of the refutation sweep (#1).

**Claim under test.** Tracee ships signatures that map onto the proxyjacking chain: `fileless_execution` (TRC-105), `dropped_executable` (TRC-1022), `stdio_over_socket` (TRC-101) (`research/ebpf-runtime-detection-single-host.md` §6).

**`dropped_executable` is not a path rule, and it is container-only.** The signature selects `magic_write` events with `Origin: "container"` and fires when the written buffer starts with ELF magic. There is no `/tmp` or `/dev/shm` list, so the `pip install --user` / `/opt` argument does not apply to it. The real limits are the ones that signature has: the payload must be ELF (any `#!/bin/sh` or Python dropper is invisible), the write must go through `magic_write`, and on an ordinary host — the corpus's stated scope — the `container` origin filter means it never fires at all.

**`fileless_execution` needs an executable to be fileless about.** `python3 -c`, `node -e`, a `dlopen`ed library: the executable is the interpreter, on disk and legitimate. `TRC-105` has nothing to match.

**`stdio_over_socket` describes a different attack.** It fires when a process `dup2`s a socket onto fd 0/1/2 — a reverse shell. A proxy relay uses the socket directly and keeps its stdio. On the population this document is about, `TRC-101` is noise.

**Re-read against the current source, the first two points get worse.** In `main` the signatures now live in `detectors/` rather than `signatures/`, and `dropped_executable` (TRC-1022) explicitly discards a write whose path fails `parsers.IsMemoryPath` — the check that recognises `memfd:` and `/dev/shm`. So the detector is not merely silent on the staging directory the corpus's own triage sweep flags first (D01); it is written to ignore it. `fileless_execution` (TRC-105) has the same shape from the other side: it matches `sched_process_exec` only for `pathname` equal to `memfd:*`, `/run/shm/*` or `/dev/shm/*`, so a payload unpacked and run from `/tmp`, `$HOME/.local/bin` or `/opt` is not "fileless" by its definition and does not fire.

**The one Tracee rule that does map is not evaluated here.** §6's policy set also enables `net_tcp_connect` / `security_socket_connect` at the LSM, which *is* relay coverage and does not depend on any signature. This issue tests the three named signatures and not that rule; the gap is the corpus presenting signatures as the coverage.

**Verdict: low for the three named signatures.** They are a dropper and reverse-shell pack. The corpus's own argument is that this threat has no malware signature, and then it lists malware signatures as the coverage — while the LSM connect event that would apply is mentioned in passing and never analysed.
