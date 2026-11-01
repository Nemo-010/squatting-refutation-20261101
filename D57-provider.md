Part of the refutation sweep (#1).

**Claim under test.** Provider-side detection is the real one: billing and abuse reports, with Oracle's idle clause reading like a detection rule (95th-percentile CPU < 20 %, network < 20 %, memory < 20 % over 7 days) and the published egress allowances making the arbitrage legible (`REPORT.md` §3.4).

**95th percentile is a duty-cycle exemption, not a threshold.** The 95th percentile ignores the top 5 % of samples. An instance that runs at full rate for one hour a day — 4.2 % of the time — has a 95th-percentile network figure near idle and pushes ~40 GB/day at 100 Mbit/s. The rule Oracle wrote to find *idle* instances is blind to a bursty one by construction, and no tuning fixes it: the statistic is the problem.

**P18 — defeat the other two columns.** 95th-percentile CPU is a busy loop (a single core at 100 % on a 1/8-OCPU micro instance is enough); memory is an allocation held resident. Both are free relative to the value of a 10 TB/month pipe.

**P19 — stay inside the allowance.** Oracle's 10 TB/month free then US$0.0085/GB makes the marginal cost of the *first* 10 TB zero. A relay capped at 8 TB/month is invisible to billing and well inside the idle rule.

**Choose a provider that does not name the activity.** The corpus quotes DigitalOcean's AUP forbidding open proxies and Tor exits while never mentioning BitTorrent. A seedbox is then a policy question that has no policy sentence to trigger.

**Churn is the business model, and it is also the bypass.** The corpus says the model is account churn and then proposes "make the account expensive to replace" as the counter. That is an economic argument, not a detection: it does not produce an alert on the night the account is used.

**Verdict: medium.** Billing is the one detector that cannot be hooked, forged or uninstalled from inside the guest — the corpus is right about that, and it is the strongest claim in the document. But the specific rule it cites (Oracle idle) is a duty-cycle exemption, and the egress allowances are budgets the attacker reads before choosing the provider.
