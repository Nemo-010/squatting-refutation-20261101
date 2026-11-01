Part of the refutation sweep (#1).

**Claim under test.** `-a always,io_uring -S openat,openat2 -F key=access_uring` covers the io_uring path, which syscall-based rules cannot see (`checks/50-proxyjack.rules`).

**It covers file opens, not the network.** The io_uring filter matches by opcode, and the rule names `openat` and `openat2`. `IORING_OP_CONNECT`, `IORING_OP_BIND` and `IORING_OP_SENDMSG` are auditable — the kernel calls `audit_uring_entry()` for them — but only if a rule names them. Under this ruleset an io_uring connect produces **zero events**. `IORING_OP_SOCKET` is not auditable at all: `opdef.c` sets `audit_skip = 1` on it.

**Data movement is not auditable at all.** `READV`, `WRITEV`, `SEND`, `RECV`, `SPLICE` and `SEND_ZC` are marked `audit_skip` in the kernel's opcode table. There is no io_uring rule that would log them, so the bytes are invisible even with a complete rule set.

**Version floor.** The uring filter needs kernel ≥ 5.16 and audit userspace ≥ 3.1. Debian 12 and Ubuntu 22.04 cannot express it, and the ruleset is shipped as-is with a comment rather than a guard.

**P06 / P07.** A ring created once and reused, or a request submitted by a helper process, moves attribution away from the payload.

**Verdict: high.** The rule closes the file-open gap the corpus was worried about and leaves the network gap — the one that matters for a relay — wide open. The fix is cheap (`-S connect,bind,sendmsg` on the uring filter, plus a separate check for socket creation, which audit cannot see); it is not in the repo.
