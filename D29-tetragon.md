Part of the refutation sweep (#1).

**Claim under test.** Tetragon does in-kernel filtering with enforcement (`Sigkill`), publishes its own overhead (1.68 % exec tracing, 5.88 % worst-case connection-rate loss), and a TracingPolicy can kill a connect to non-private space (`research/ebpf-runtime-detection-single-host.md` §5).

**This is the strongest on-host control in the corpus, and it is bypassed operationally rather than technically.**

**P20.** The example policy is `tcp_connect`. UDP is a separate policy (`5.2`), and a QUIC/hysteria2 payload never triggers the TCP one. Writing both is possible; the repo ships one.

**P05.** `tcp_connect` is a kernel function, so io_uring's connect *does* reach it — this is Tetragon's advantage over auditd. But the per-syscall selectors (`matchArgs` on `sock` fields) cannot see a peer that is a CDN, and the kill happens after the connection is attempted (repeated retries are a self-DoS).

**Enforcement is kill, not block.** A supervisor restarts the payload; the tunnel is down for a second per attempt. The control costs the attacker nothing and produces a log line each time, which is an alert the operator must then tune.

**Root stops it.** `systemctl stop tetragon`, an edited policy file under `/etc/tetragon/tetragon.tp.d/`, or a competing eBPF program. Tetragon reports its own overhead but not its own liveness; nothing in the corpus verifies the daemon is running at the moment of the incident.

**Kernel floor.** 4.19 minimum, 6.1 for `fentry`, 5.3 for full paths, and Lockdown `confidentiality` makes BPF loading fail with `EPERM`. On a hardened host the tool does not start.

**What is actually unique is enforcement.** Tetragon can filter in-kernel and `Sigkill` on a match, and its policies are data rather than code. It is not unique in *seeing* container traffic — kprobes are global to the kernel, so bpftrace and Tracee see containers too — and a pre-existing flow produces no `tcp_connect`, here as in D25.

**Verdict: low — the best available.** If the policy covers both TCP and UDP, the daemon is alive, and the kernel allows BPF, this is the only tier that can block rather than describe, and the only one whose policy an operator can review as data. Pre-existing flows are invisible to it, and everything above — the policy, the daemon, the kernel setting — is one root command away from being wrong.
