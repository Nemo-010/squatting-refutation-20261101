Part of the refutation sweep (#1).

**Claim under test.** Six probes cover ~everything: `execve`, `connect`, `socket`, `openat("/dev/net/tun")`, `memfd_create`, `setsockopt` (`research/ebpf-runtime-detection-single-host.md` §1).

**Each probe names one syscall that has a sibling.**

- `tracepoint:syscalls:sys_enter_execve` does not fire for `execveat`. `fexecve()` and `execveat(fd, "", ..., AT_EMPTY_PATH)` — how a memfd payload actually runs — go through `sys_enter_execveat`. P02 and D20 in one line.
- `openat("/dev/net/tun")` does not fire for `openat2`, which is a separate tracepoint. glibc has no `openat2` wrapper, so this is not a libc habit — it is a payload calling `syscall(SYS_openat2, ...)` itself, or opening the device through a path form the equality filter misses.
- `memfd_create` is P02-avoidable; the probe only sees the technique, not the payload.
- `setsockopt` for `SO_MARK`/`IP_TRANSPARENT`/`SO_BINDTODEVICE` fires only on those options. A relay that sets `TCP_ULP`, `TCP_CONGESTION` or nothing at all is silent.
- `socket(AF_INET, SOCK_STREAM)` is what every client does; it is a volume probe, not a discriminator.

**A whole traffic family is only volume.** No probe in the six sees `sendto`/`sendmsg` on an unconnected UDP socket (P20), so a QUIC/hysteria2/WireGuard/`dnstt` payload produces one `socket(AF_INET, SOCK_DGRAM)` and nothing else. The "covers ~everything" claim fails on a family, not just on two syscall variants.

**P05.** None of the tracepoints fire for io_uring submissions. A ring doing `SOCKET`/`CONNECT`/`SENDMSG` is invisible to every probe in this section — the same gap as D23, from the other side.

**P10.** Probes are global to the kernel, so containers *are* covered — this is a real strength, and the one place the eBPF tier beats `ss`. The corpus does not make that comparison, and it should.

**Verdict: medium.** As a starting set it is reasonable. As "covers ~everything" it is a list of syscalls, each with a documented variant that evades it, and the strongest probe (kprobe at a convergence point rather than a tracepoint at an entry point) is the one the corpus uses for `tcp_connect` and not for the rest.
