# Assignment 05 — Nnanna Arnold Muoneke

**GitHub username:** arnold-007
**Date completed:** 2026-07-19

## 1. Answers to the 10 questions

**Q1 — control-plane components + per-node bucket:** 

The kube-apiserver is the comoponent that plays the receptionist role, taking all your commands/requests to kubernetes and forwarding them to the ectd which is single source of truth database that stores every info about the k8s infrasctructure which you have set up.
Based on the instructions/wishes communicated to kubernetes, the kube-scheduler then tries to matchmake your desired state with resources available in the cluster. It then relays this matchmaking info back to the kube-apiserver.
Finally we have the kube-controller-manager which plays the babysitter role by monitoring the state of the cluster and making necessary adjustments to match your desired request.

kindnet and kube-proxy both run on each node with each pod having different ID Suffixes. A DaemonSet ensures a copy of a pod runs on all 3 nodes in this example. Seeing that both kindnet and kube-proxy are both node-level networking components, a DaemonSet actively manages their creation and destruction based on cluster activity (node addition/removal).

**Q2 — static pods + bootstrap chicken-and-egg:** 

The kubelet has a built-in static pod feature that watches the /etc/kubernetes/manifests directory by default and can read YAML/JSONs and through them creates pods (Kind docker containers) directly using the container runtime (containerd etc). After creating the static pod for API-server you can now create other components normally therefore breaking the circular deadlock. 

kubectl delete only deletes a static pod's mirror object permanently from the API Server if you delete the single source of truth which is the manifest. The manifest gives the kubelet ultimate power over the API Server allowing it to recreate the pod and forcing the API Server's hand to make a new mirror object.

**Q3 — etcd quorum + stateless API server:** 

Based on quorum math alone: a 1 member cluster would need a quorum of 1 node meanwhile 2-member cluster needs a quorum of (2/2 +1)=2 nodes. So if a 1-member cluster has afault it is basically gone otherwise it works just fine. A 2-member cluster has to be a perfect two out two healthy nodes because once one node dies the cluster is as well dead. For double the resources you risk the same probability of a dead cluster as the 1-member cluster.

With a stateless API server, if etcd's data is destroyed, the only survivors would be the Kubelet and every thing that runs with it (containerd) or is maintained by it (static pods and its affiliated pods)

**Q4 — contexts + context-drift accident:** 

The reason why `kubectl get pods` reacted differently is because the config of the k8s-lab-system context had the field `namespace: kube-system` hard-coded in while kind-k8s-lab does not have that field and therefore implicitly refers to the default namespace.

Context drift can cause a command to modify or restart a workload in the wrong cluster or namespace.  If for example an engineer intends to restart an application in a namespace but is using a context whose default namespace is totally different different this could lead to all sorts of issues and could even be difficult to resolve or troubleshoot.
Before running a risky command, one could check or print out which context and default namespace are active, then verify if it matches with the intended target.

..
**Q5 — request flow authn/authz/admission/persist + dry-run limits:** ...


**Q6 — observe/diff/act mapping:** 
The ReplicaSet controller observed the scaling to 3 pods and later that one of the three pods it owned had been deleted (pod 2t7xz go Terminating → Completed → gone). It compared the desired count (3, set by kubectl scale) against the actual count (2) and found a DIFF of one pod. It then acted by creating one new pod (d7svk), which appeared almost immediately. The desired state never changed AND only the observed state drifted. So the controller simply closed the gap.

...

**Q7 — scheduler-down blast radius:** 
With the scheduler absent, the ReplicaSet controller still created the two new pods since it only needs the API server, but they are Pending because the scheduler's not there to write spec.nodeName to bind each pod to a node. Already-running pods kept serving traffic normally, since kubelet and the dataplane don't depend on the control plane for that (kind of brings it back to the chicken and egg question). This shows the data plane is mostly independent from the control plane. The control plane is needed to change desired state or place new pods, but not to keep existing traffic flowing.
...


**Q8 — kubelet as systemd + kube-proxy + pause container:** 
(a) kubelet runs as a systemd service because it is the component that creates Pods so if it ran as a Pod itself, it would have to create itself (chicken-and-egg)., so systemd must start it before any Pod exists.

(b) kube-proxy only writes iptables rules and the Linux kernel's netfilter matches those rules and rewrites the destination to a real Pod IP (DNAT).

(c) The pause container holds the shared network namespace all the Pod's containers join. With it, the containers would have shared IP and localhost, exactly the --network container:api mechanism from Assignment 04. Without it, you cant address the containers as a single pod.

...

**Q9 — kubectl top + aggregation layer:** 
`get pods` asks for `v1 group` which is served **natively by the apiserver** from etcd; ` kubectl top` asks for `metrics.k8s.io`, which is an outside APIService the apiserver **proxies via the aggregation layer** to an external metrics-server. And on a fresh cluster metrics-server isn't installed, so that leads to the error "Metrics API not available" and then remedied by installing metrics; native resources work because they need no add-on.
...

**Q10 — three ways to get an image + which for scripts:** ...
 1) describe + grep
kubectl describe deployment drill | grep -i image

 2) jsonpath
kubectl get deployment drill -o jsonpath='{.spec.template.spec.containers[*].image}{"\n"}'

 3) yaml + grep
kubectl get deployment drill -o yaml | grep -i 'image:'

 (bonus) get -o wide
kubectl get deployment drill -o wide

I would use any of the grep operations or -o wide for easy viewing sake but downside is too much information which I would have to get rid off for scripting. So ultimately, jsonpath gets straight to the point without the need to parse for "image" or filter other things out.
...

## 2. Cluster survey

Paste the output of:

- `kubectl get nodes -o wide`
  
 ` NAME                    STATUS   ROLES           AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                       KERNEL-VERSION                         CONTAINER-RUNTIME
k8s-lab-control-plane   Ready    control-plane   6d10h   v1.36.1   172.18.0.2    <none>        Debian GNU/Linux 13 (trixie)   5.14.0-687.29.1.el9_8.x86_64 (amd64)   containerd://2.3.1
k8s-lab-worker          Ready    <none>          6d10h   v1.36.1   172.18.0.3    <none>        Debian GNU/Linux 13 (trixie)   5.14.0-687.29.1.el9_8.x86_64 (amd64)   containerd://2.3.1
k8s-lab-worker2         Ready    <none>          6d10h   v1.36.1   172.18.0.4    <none>        Debian GNU/Linux 13 (trixie)   5.14.0-687.29.1.el9_8.x86_64 (amd64)   containerd://2.3.1
`


- `kubectl get pods -n kube-system -o wide` (with your three-bucket classification)
  

Pods running on control-plane:

`
NAME                                            READY   STATUS    RESTARTS   AGE   IP           NODE                    NOMINATED NODE   READINESS GATES
etcd-k8s-lab-control-plane                      1/1     Running   0          5h5m   172.18.0.2   k8s-lab-control-plane   <none>           <none>
kube-apiserver-k8s-lab-control-plane            1/1     Running   0          5h5m   172.18.0.2   k8s-lab-control-plane   <none>           <none>
kube-controller-manager-k8s-lab-control-plane   1/1     Running   0          5h5m   172.18.0.2   k8s-lab-control-plane   <none>           <none>
kube-scheduler-k8s-lab-control-plane            1/1     Running   0          5h5m   172.18.0.2   k8s-lab-control-plane   <none>           <none>
`

Pods running on every node:

`
NAME                                            READY   STATUS    RESTARTS   AGE   IP           NODE                    NOMINATED NODE   READINESS GATES
kindnet-2hl86                                   1/1     Running   0          5h5m   172.18.0.2   k8s-lab-control-plane   <none>           <none>
kindnet-9cn8g                                   1/1     Running   0          5h5m   172.18.0.3   k8s-lab-worker          <none>           <none>
kindnet-tr67s                                   1/1     Running   0          5h5m   172.18.0.4   k8s-lab-worker2         <none>           <none>
kube-proxy-c4v49                                1/1     Running   0          5h5m   172.18.0.2   k8s-lab-control-plane   <none>           <none>
kube-proxy-kqgps                                1/1     Running   0          5h5m   172.18.0.3   k8s-lab-worker          <none>           <none>
kube-proxy-xnskl                                1/1     Running   0          5h5m   172.18.0.4   k8s-lab-worker2         <none>           <none>
`

Anything else:

`NAME                                            READY   STATUS    RESTARTS   AGE   IP           NODE                    NOMINATED NODE   READINESS GATES
coredns-589f44dc88-vzsnw                        1/1     Running   0          5h5m   10.244.0.3   k8s-lab-control-plane   <none>           <none>
coredns-589f44dc88-zv8jc                        1/1     Running   0          5h5m   10.244.0.2   k8s-lab-control-plane   <none>           <none>
`




- `docker exec k8s-lab-control-plane ls /etc/kubernetes/manifests`
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml


## 3. Evidence

Paste command + output (trim to the relevant lines):

- Part 2.2 — the etcd key for your `etcd-canary` pod
Command:  `kubectl exec -n kube-system etcd-k8s-lab-control-plane --   etcdctl --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key   get /registry/pods/default/etcd-canary --prefix --keys-only`

Output: `/registry/pods/default/etcd-canary`

- Part 3.2 — `kubectl get pods` under the `k8s-lab-system` context (no -n flag)

[root@localhost ~]# kubectl config use-context k8s-lab-system
`Switched to context "k8s-lab-system".`

[root@localhost ~]# kubectl get pods
`NAME                                            READY   STATUS    RESTARTS   AGE
coredns-589f44dc88-l5dvc                        1/1     Running   0          28s
coredns-589f44dc88-l5znz                        1/1     Running   0          28s
etcd-k8s-lab-control-plane                      1/1     Running   0          36s
kindnet-bgdcx                                   1/1     Running   0          24s
kindnet-ltrlz                                   1/1     Running   0          24s
kindnet-zkrgd                                   1/1     Running   0          28s
kube-apiserver-k8s-lab-control-plane            1/1     Running   0          36s
kube-controller-manager-k8s-lab-control-plane   1/1     Running   0          36s
kube-proxy-4vtzs                                1/1     Running   0          24s
kube-proxy-dbrjr                                1/1     Running   0          24s
kube-proxy-ppdgc                                1/1     Running   0          28s
kube-scheduler-k8s-lab-control-plane            1/1     Running   0          36s
`
- Part 4.2 — the request line(s) from `kubectl apply -v=8`
- Part 5.1 — the `-w` output showing the deleted pod replaced
  `NAME                     READY   STATUS         RESTARTS       AGE
flow-demo-84745775cb-2t7xz   1/1     Running         0             7m52s
flow-demo-84745775cb-cgfs6   0/1     Pending         0             0s
flow-demo-84745775cb-cgfs6   0/1     Pending         0             0s
flow-demo-84745775cb-xd8s5   0/1     Pending         0             0s
flow-demo-84745775cb-xd8s5   0/1     Pending         0             0s
flow-demo-84745775cb-cgfs6   0/1     ContainerCreating   0          0s
flow-demo-84745775cb-xd8s5   0/1     ContainerCreating   0          0s
flow-demo-84745775cb-2t7xz   1/1     Terminating         0          11m
flow-demo-84745775cb-d7svk   0/1     Pending             0          0s
flow-demo-84745775cb-2t7xz   1/1     Terminating         0          11m
flow-demo-84745775cb-d7svk   0/1     Pending             0          0s
flow-demo-84745775cb-2t7xz   0/1     Completed           0          11m
flow-demo-84745775cb-2t7xz   0/1     Completed           0          11m
flow-demo-84745775cb-2t7xz   0/1     Completed           0          11m
flow-demo-84745775cb-xd8s5   0/1     ContainerCreating   0          1s
flow-demo-84745775cb-cgfs6   0/1     ContainerCreating   0          1s
flow-demo-84745775cb-d7svk   0/1     ContainerCreating   0          1s
flow-demo-84745775cb-xd8s5   1/1     Running             0          1s
flow-demo-84745775cb-d7svk   0/1     ContainerCreating   0          1s
flow-demo-84745775cb-cgfs6   1/1     Running             0          4s
flow-demo-84745775cb-d7svk   1/1     Running             0          4s
`
- Part 5.2 — the stuck pod: `Pending` phase + empty `spec.nodeName`
  `NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE              NOMINATED NODE   READINESS GATES
flow-demo-84745775cb-cgfs6   1/1     Running   0          53m   10.244.1.2   k8s-lab-worker2   <none>           <none>
flow-demo-84745775cb-d7svk   1/1     Running   0          53m   10.244.1.3   k8s-lab-worker2   <none>           <none>
flow-demo-84745775cb-qscqv   0/1     Pending   0          0s    <none>       <none>            <none>           <none>
flow-demo-84745775cb-xd8s5   1/1     Running   0          53m   10.244.2.3   k8s-lab-worker    <none>           <none>
flow-demo-84745775cb-xl86w   0/1     Pending   0          0s    <none>       <none>            <none>           <none>
`
- Part 5.3 — the same pods `Running` after the scheduler returned
  `NAME                         READY   STATUS    RESTARTS   AGE     IP           NODE              NOMINATED NODE   READINESS GATES
flow-demo-84745775cb-cgfs6   1/1     Running   0          3h49m   10.244.1.2   k8s-lab-worker2   <none>           <none>
flow-demo-84745775cb-d7svk   1/1     Running   0          3h49m   10.244.1.3   k8s-lab-worker2   <none>           <none>
flow-demo-84745775cb-qscqv   1/1     Running   0          175m    10.244.2.5   k8s-lab-worker    <none>           <none>
flow-demo-84745775cb-xd8s5   1/1     Running   0          3h49m   10.244.2.3   k8s-lab-worker    <none>           <none>
flow-demo-84745775cb-xl86w   1/1     Running   0          175m    10.244.2.4   k8s-lab-worker    <none>           <none>
`
- Part 7.3 — outputs of drills (a)–(d)
  
(a)
[root@2001-4dd4-4c98-0-be24-11ff-fedb-8ed0 ~]# kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.spec.nodeName}{"\n"}{end}'


`default drill-84799678b7-6vmc6  k8s-lab-worker
default drill-84799678b7-h2ncn  k8s-lab-worker2
default flow-demo-84745775cb-cgfs6      k8s-lab-worker2
default flow-demo-84745775cb-d7svk      k8s-lab-worker2
default flow-demo-84745775cb-qscqv      k8s-lab-worker
default flow-demo-84745775cb-xd8s5      k8s-lab-worker
default flow-demo-84745775cb-xl86w      k8s-lab-worker
kube-system     coredns-589f44dc88-l5dvc        k8s-lab-control-plane
kube-system     coredns-589f44dc88-l5znz        k8s-lab-control-plane
kube-system     etcd-k8s-lab-control-plane      k8s-lab-control-plane
kube-system     kindnet-bgdcx   k8s-lab-worker2
kube-system     kindnet-ltrlz   k8s-lab-worker
kube-system     kindnet-zkrgd   k8s-lab-control-plane
kube-system     kube-apiserver-k8s-lab-control-plane    k8s-lab-control-plane
kube-system     kube-controller-manager-k8s-lab-control-plane   k8s-lab-control-plane
kube-system     kube-proxy-4vtzs        k8s-lab-worker2
kube-system     kube-proxy-dbrjr        k8s-lab-worker
kube-system     kube-proxy-ppdgc        k8s-lab-control-plane
kube-system     kube-scheduler-k8s-lab-control-plane    k8s-lab-control-plane
local-path-storage      local-path-provisioner-855c7b7774-dwljz k8s-lab-control-plane`


(b)
[root@2001-4dd4-4c98-0-be24-11ff-fedb-8ed0 ~]# kubectl get pods -n kube-system -o jsonpath='{.items[*].spec.containers[*].image}' | tr ' ' '\n' | sort -u
`
docker.io/kindest/kindnetd:v20260528-9350166c
registry.k8s.io/coredns/coredns:v1.14.2
registry.k8s.io/etcd:3.6.8-0
registry.k8s.io/kube-apiserver:v1.36.1
registry.k8s.io/kube-controller-manager:v1.36.1
registry.k8s.io/kube-proxy:v1.36.1
registry.k8s.io/kube-scheduler:v1.36.1`



(c) 
[root@2001-4dd4-4c98-0-be24-11ff-fedb-8ed0 ~]# kubectl get pods -A --field-selector status.phase!=Running
`No resources found`



  (d)
[root@2001-4dd4-4c98-0-be24-11ff-fedb-8ed0 ~]# kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.allocatable.memory}{"\n"}{end}'
`k8s-lab-control-plane   2777328Ki
k8s-lab-worker  2777328Ki
k8s-lab-worker2 2777328Ki
`
  
- Part 7.4 — `kubectl top nodes` working after the metrics-server fix

  

`[root@2001-4dd4-4c98-0-be24-11ff-fedb-8ed0 ~]# kubectl top nodes
`
`NAME                    CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
k8s-lab-control-plane   61m          3%     832Mi           30%
k8s-lab-worker          20m          1%     363Mi           13%
k8s-lab-worker2         18m          0%     326Mi           12%
`

## 4. One trade-off I had to make

(2–4 sentences. Pick one: imperative vs declarative for the drills, patching metrics-server
vs re-writing its manifest, keeping vs deleting the cluster between assignments, etc.
Explain what you chose and what the other option would have bought you.)

Always having to use the full path of kind to create a cluster: /usr/local/bin/kind create cluster --config kind-basic.yaml --name k8s-lab

## 5. One thing I'm still unsure about

Why does coredns from kube-system and local-path-provisioner pods both have 10.244.0.3 and 10.244.0.4 as IPs?
