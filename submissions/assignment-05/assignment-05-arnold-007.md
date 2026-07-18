# Assignment 05 — <Your Name>

**GitHub username:** <your-username>
**Date completed:** YYYY-MM-DD

## 1. Answers to the 10 questions

**Q1 — control-plane components + per-node bucket:** ...
**Q2 — static pods + bootstrap chicken-and-egg:** ...
**Q3 — etcd quorum + stateless API server:** ...
**Q4 — contexts + context-drift accident:** ...
**Q5 — request flow authn/authz/admission/persist + dry-run limits:** ...
**Q6 — observe/diff/act mapping:** ...
**Q7 — scheduler-down blast radius:** ...
**Q8 — kubelet as systemd + kube-proxy + pause container:** ...
**Q9 — kubectl top + aggregation layer:** ...
**Q10 — three ways to get an image + which for scripts:** ...

## 2. Cluster survey

Paste the output of:

- `kubectl get nodes -o wide`
- `kubectl get pods -n kube-system -o wide` (with your three-bucket classification)
- `docker exec k8s-lab-control-plane ls /etc/kubernetes/manifests`

## 3. Evidence

Paste command + output (trim to the relevant lines):

- Part 2.2 — the etcd key for your `etcd-canary` pod
- Part 3.2 — `kubectl get pods` under the `k8s-lab-system` context (no -n flag)
- Part 4.2 — the request line(s) from `kubectl apply -v=8`
- Part 5.1 — the `-w` output showing the deleted pod replaced
- Part 5.2 — the stuck pod: `Pending` phase + empty `spec.nodeName`
- Part 5.3 — the same pods `Running` after the scheduler returned
- Part 7.3 — outputs of drills (a)–(d)
- Part 7.4 — `kubectl top nodes` working after the metrics-server fix

## 4. One trade-off I had to make

(2–4 sentences. Pick one: imperative vs declarative for the drills, patching metrics-server
vs re-writing its manifest, keeping vs deleting the cluster between assignments, etc.
Explain what you chose and what the other option would have bought you.)

## 5. One thing I'm still unsure about

(One sentence. Goes to office hours.)
