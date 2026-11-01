Part of the refutation sweep (#1).

**Claim under test.** Sweep pods for `hostNetwork`/`hostPID`/`privileged`/`hostPath`, inventory DaemonSet images, find who holds `nodes/proxy`/`pods/exec`, read audit logs, watch `kubelet_http_requests_total`, and diff `crictl ps` against `kubectl get pods` (`research/kubernetes-and-container-squatting.md` §8–§10).

**Reuse an object instead of creating one.** The sweep looks for *suspicious pods*. Editing an existing DaemonSet's image tag — `node-exporter` becomes a proxy — produces a diff of one field on an object that has been there for months, and the image name is whatever the attacker tags it. Diffing against last week's output catches it; a one-shot sweep does not.

**`hostNetwork` pods are invisible to container network metrics.** cAdvisor's `container_network_transmit_bytes_total` comes from the pod's veth. A `hostNetwork: true` pod has no veth; its traffic is the host's. The number that would show the egress is not a pod metric at all, and the corpus recommends cAdvisor for exactly this measurement.

**Static pods have no mirror object.** The corpus notes it; the consequence is that a payload delivered as a static pod (file in `/etc/kubernetes/manifests`) is absent from `kubectl get pods -A` and from the DaemonSet inventory. Only the node's `/proc` and `/var/lib/kubelet` show it.

**Orphans can be named to match.** `crictl ps` vs the API diff works because names differ. A container started directly through the runtime can reuse the sandbox's name and labels; the diff then reports nothing new.

**A valid identity defeats the audit trail.** `pods/exec` records the connection, not argv. An attacker with a stolen service-account token produces audit lines indistinguishable from an operator's, and `nodes/proxy` — the corpus's own "equivalent to exec on every container, not in the API audit log" — produces none at all.

**Verdict: medium.** The object queries are a good inventory and the RBAC check is the best item here. Everything that depends on "the squat created a new object" is defeated by reusing one, and everything that depends on per-pod network metrics is defeated by the one flag the corpus already flags as normal (`hostNetwork`).
