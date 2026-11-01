Part of the refutation sweep (#1).

**Claim under test.** `-S memfd_create` catches anonymous-memory payloads; the follow-on `execveat` cannot be matched because its flags live in `a4` and audit exposes only `a0`–`a3`, so the `/proc/<pid>/exe` sweep for `/memfd:` substitutes (`research/auditd-proc-detection.md` §3).

**P02 — no memfd.** An interpreter (`python3 -c`, `node -e`), a `dlopen`ed library, or a JIT-compiled region executes code with no new image. There is no `memfd_create`, no `execve`, and no `/memfd:` to sweep.

**P01 — a real file in a directory nobody lists.** The whole point of a memfd is to avoid a path; `$HOME/.local/bin` achieves the same and survives a reboot.

**The follow-on is matchable, just not by flag.** `-S execveat` with no argument filter logs every `execveat`, `AT_EMPTY_PATH` included; what cannot be matched is `a4`, i.e. the flag itself. So the rule the corpus wants is `-a always,exit -F arch=b64 -S execveat -k memfd_exec`, at the cost of every `execveat` on the host. The ruleset ships no `execveat` rule at all, and `audit-sweep.sh` greps a `memfd_exec` key that nothing sets.

**P06.** The memfd is created by process A and passed to process B with `SCM_RIGHTS`; `memfd_create` fires once, on A, whose pid may already be gone. B does the exec, and no rule names `execveat`.

**The `/proc` substitute is the same check as D01**, with the same blind spots: non-root reads fail, hooks rewrite the link, and `/memfd:` is exactly the string an actor avoids.

**Verdict: low.** Both halves are optional for the attacker: the rule covers a technique a script can replace, and the substitute depends on a string the payload chooses. The corpus is honest that `a4` cannot be matched; it is not honest that the same exec could be logged by syscall name, which is cheaper than the `/proc` sweep.
