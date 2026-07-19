# Assignment 05 — Nnanna Arnold Muoneke

**GitHub username:** arnold-007
**Date completed:** 2026-07-19

## 1. Answers to the 10 questions

**Q1 — control-plane components + per-node bucket:** 
### Pods running on control-plane:
`NAME                                            READY   STATUS    RESTARTS   AGE   IP           NODE                    NOMINATED NODE   READINESS GATES
etcd-k8s-lab-control-plane                      1/1     Running   0          23h   172.18.0.2   k8s-lab-control-plane   <none>           <none>
kube-apiserver-k8s-lab-control-plane            1/1     Running   0          23h   172.18.0.2   k8s-lab-control-plane   <none>           <none>
kube-controller-manager-k8s-lab-control-plane   1/1     Running   0          23h   172.18.0.2   k8s-lab-control-plane   <none>           <none>
kube-scheduler-k8s-lab-control-plane            1/1     Running   0          23h   172.18.0.2   k8s-lab-control-plane   <none>           <none>
`
### Pods running on every node:
`NAME                                            READY   STATUS    RESTARTS   AGE   IP           NODE                    NOMINATED NODE   READINESS GATES
kindnet-d59f8                                   1/1     Running   0          23h   172.18.0.4   k8s-lab-worker2         <none>           <none>
kindnet-r4khp                                   1/1     Running   0          23h   172.18.0.3   k8s-lab-worker          <none>           <none>
kindnet-r8qw6                                   1/1     Running   0          23h   172.18.0.2   k8s-lab-control-plane   <none>           <none>
kube-proxy-b68tb                                1/1     Running   0          23h   172.18.0.4   k8s-lab-worker2         <none>           <none>
kube-proxy-bbdjg                                1/1     Running   0          23h   172.18.0.2   k8s-lab-control-plane   <none>           <none>
kube-proxy-zbr79                                1/1     Running   0          23h   172.18.0.3   k8s-lab-worker          <none>           <none>
`
### Anything else:
`NAME                                            READY   STATUS    RESTARTS   AGE   IP           NODE                    NOMINATED NODE   READINESS GATES
coredns-589f44dc88-67sbq                        1/1     Running   0          23h   10.244.0.3   k8s-lab-control-plane   <none>           <none>
coredns-589f44dc88-7j5bq                        1/1     Running   0          23h   10.244.0.2   k8s-lab-control-plane   <none>           <none>
`
kube-apiserver
etcd
kube-scheduler
kube-controller-manager...
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
