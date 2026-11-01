Part of the refutation sweep (#1).

**Claim under test.** A notebook leaves a durable trail: `history.sqlite` with every cell, `kernel-<id>.json` with five ZMQ ports and the HMAC key, `jpserver-<pid>.json` with the token, and `.ipynb` outputs with stdout and base64 blobs (`research/jupyter-multitenant-detection.md` §2).

**Terminals do not write IPython history.** The corpus notes that a terminal tab is a separate `bash` child of the single-user server. Commands typed there execute outside the kernel, so `history.sqlite` records nothing — and a terminal is the shortest path from "I have the token" to "I have a shell".

**The database is in the user's home and is user-writable.** `IPYTHONDIR` relocates it (the same attacker-settable mechanism the corpus flags for `JUPYTER_RUNTIME_DIR`); `history.sqlite` can be truncated or deleted; a kernel started with `store_history=False` writes no rows; and `subprocess.run(...)` from a cell records one line that says nothing about what the script did.

**Runtime files can be elsewhere and gone.** `JUPYTER_RUNTIME_DIR` is honoured, and the files are deleted on clean shutdown. The corpus says to check `/proc/<pid>/environ` first — which is correct, and also the reason the artifact hunt fails when the process is no longer running, i.e. whenever the operator looks after the fact.

**`.ipynb` outputs only exist if a notebook was used.** A payload delivered through the terminal, through `pkill`-style one-liners, or through a Jupyter *server extension* leaves no notebook at all.

**Verdict: medium.** These artifacts are excellent for attributing what a *careless* user did in the UI, and the corpus's triage loop is genuinely useful. They are not a detector: the interesting session is the one that used the terminal, cleared the history, and deleted the runtime directory — all of which are ordinary user actions.
