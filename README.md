# Refuting the squatting corpus

**Fifty-nine adversarial issues against the detections in
[squatting-20261031](https://github.com/Nemo-010/squatting-20261031) — one per detection family, each
naming the cheapest way past it.**

The corpus describes how a borrowed host is turned into a proxy, a relay, a seedbox or a notebook,
and how to find that on one box with no budget. This document does the opposite: it takes every
detection the corpus recommends, states the claim it rests on, and looks for the move that makes it
fire nothing. Where the move is free, the issue says so. Where the squat genuinely cannot be
distinguished from legitimate use, the issue says that instead of inventing a residual signal.

Start with **[index.md](index.md)** — it holds the twenty-six cross-cutting bypass primitives, the
index of all fifty-nine issues with a verdict each, and the short list of what survives.

## The result in one paragraph

The verdicts are not close. Thirty-four of the fifty-nine carry a `medium` verdict — the check fires on
a default deployment and is erased by one flag — and eight more are `high` or `medium-high`, meaning
the bypass is free or nearly so. Seven are `mixed`: one half of the check survives, the other does not,
and the issue says which is which. Only seven are `low`, and one of those, Tetragon, is `low` in the
other sense — it is the best of its tier, and the number describes what a bypass would cost rather
than how little the check is worth. What survives is small and dull: whole-host egress summed against
what the host is supposed to be doing, the provider's own bill and flow records, one egress allow-list
that does not include a resolver, a handful of cheap exact-format checks that cost nothing and never
lie, and — in one tier only — enforcement rather than observation. Everything else measures the
packaging of the workload, not its intent, which is why a host used for the purpose it was provisioned
for produces no anomaly in any of it.

## Method

Every issue is the same shape, so the issues can be compared without reading them twice:

- **Claim under test** — the detection as the corpus states it, with the file and section it comes
  from, so the claim can be checked rather than taken on trust.
- **The bypasses** — each one names its prerequisite. "Root can do anything" is not a finding and is
  not counted. A bypass is real only if it is cheaper than the detection.
- **Verdict** — `high` when the bypass is free or one ordinary design choice away; `medium` when it
  costs the squatter a specific but unremarkable move; `low` when the check holds for the population
  the corpus is about; `indistinguishable` when no signal exists at all. Two issues test no check and
  carry no verdict.
- **`keep`** — a separate recommendation: run it, use it as a lead for a different question, or drop
  it from a squatting checklist.

The issues are descriptive. They contain no payloads and no working bypass code: the mechanism is
named, and the reader is expected to be the person who has to fix the detector.

## What was re-checked first-hand (2026-11-01)

Load-bearing numbers were checked against the source, not against the corpus's summary of it. The
re-checks that changed a file:

- **`systemd.path`** — both trigger-rate-limit defaults and the "placed into a failure mode, and will
  not watch the paths anymore until restarted" wording are verbatim in `systemd.path(7)` for systemd
  262, added in 250. `DirectoryNotEmpty=` evaluates `dir_is_empty(path, ignore_hidden_or_backup=true)`
  in `src/core/path.c`, which is the second half of the dotfile bypass in D46.
- **Tracee** — `dropped_executable` (TRC-1022) now lives in `detectors/` and discards any write whose
  path `parsers.IsMemoryPath` recognises, so `memfd:` and `/dev/shm` are excluded by construction;
  `fileless_execution` (TRC-105) matches `sched_process_exec` only for `memfd:*`, `/run/shm/*` and
  `/dev/shm/*` (D30).
- **The Tor bulk exit list** — 1,228 lines on 2026-11-01 against the corpus's 1,225 on 2026-10-31, and
  still zero lines containing a colon, so the IPv4-only finding holds (D39).
- **systemd `IPAccounting`** — systemd issue [#34775](https://github.com/systemd/systemd/issues/34775)
  reproduces the 10 KB → `11001` byte overcount/undercount on GSO packets (D12).
- **Kernel** — `tun_chr_compat_ioctl` exists in `drivers/net/tun.c` (D18); `modules_disabled` "cannot
  be set back to false" in `Documentation/admin-guide/sysctl/kernel.rst` (D22).
- **`checks/triage.sh`** — `readlink … || continue` suppresses the EACCES on another user's
  `/proc/<pid>/exe`, and the "not root" note is printed before the sweep rather than counted into the
  result (D01).
- **Citations** — every numeric section reference from an issue into `REPORT.md` or `research/*.md`
  was resolved against that file's headings; the one that did not resolve (`§5.9` in D43) is corrected.

Numbers that carry a timestamp were measured on 2026-11-01 and will drift.

## Layout

```
index.md                 primitives P01–P26, the 59-row index, and what survives
D01 … D59                one issue per detection family, in the corpus's reading order
```

The corpus itself — report, fifteen deep dives, and the runnable checks — is
[squatting-20261031](https://github.com/Nemo-010/squatting-20261031).

## License

0BSD. See [LICENSE](LICENSE).
