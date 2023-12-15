---
title: "K8s - Solve pod is always pending"
date: 2023-12-15T14:38:06+08:00
lastmod: 2023-12-15T14:38:06+08:00
draft: false
tags: ["technology", "k8s"]
categories: ["k8s"]
author: "大白猫"
---

## Statement

I'm using *colima* & *kind* & *kubebuilder* to test a operator project in my local env. The way I set up a local kind cluster is [Contour_Create_a_Kind_Cluster](https://projectcontour.io/docs/1.27/guides/kind/). The cluster looks like:

```bash
>> kubectl get node
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   24h   v1.27.3
kind-worker          Ready    <none>          24h   v1.27.3
```

Then use `make run` to start and deploy a service. However, when I used `kubectl get all -n ns` to check the status of the service, I found that there is 0/1 pod READY and its STATUS is always *pending*. Also, the replicaset and deployment is not ready.

## Trouble-shooting Step

Describe the pod and check the events:

``` bash
kubectl describe pod pod-name -n ns
```

Find the events:

```bash
Events:
Type Reason Age From Message
---- ------ ---- ---- -------
Warning  FailedScheduling  6m21s  default-scheduler  0/2 nodes are available: 1 Insufficient cpu, 1 Insufficient memory, 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/2 nodes are available: 1 No preemption victims found for incoming pod, 1 Preemption is not helpful for scheduling..
```

It means there is no node available to schedule the pod and it gives some reasons. In my case, it indicates three issues:

1. The node doesn't have sufficient cpu.
2. The node doesn't have sufficient memory.
3. The node has taint, and the pod has no related taint tolerations.

So check the node:

```bash
kubectl describe node kind-control-plane
kubectl describe node kind-worker
```

For issue#1 and issue#2, from the describe of node (both `kind-control-plane` and `kind-worker`), I can see the cpu number is 2 and the memory is 1941184Ki (about 2 GB) in *Capacity* and *Allocatable* part. It is limited by *colima* (some cases from google say that *kind* cannot add resource limitation, so it limited by *colima* vm). And in *Conditions* part, there are MemoryPressure, DiskPressure and PIDPressure. We can edit *colima* config by this command line and restart it.

```bash
colima start --edit
```

Expand the cpu to 4 and the memory to 8 GB. Then the cpu and memory limitation is updated in node. (But the MemoryPressure, DiskPressure and PIDPressure still exist with False, why?)

For issue#3, in node `kind-control-plane` there is a taint: `Taints: node-role.kubernetes.io/control-plane:NoSchedule`. There are two ways to resolve it:

* Delete the taint in node (Not recommend)
  ``` bash
  >> kubectl taint nodes kind-control-plane node-role.kubernetes.io/control-plane-
  node/kind-control-plane untainted
  ```

  (mind that there is a `-` at the end)

* Add taint tolerance on pod (I never tried)
  ```bash
  kubectl edit deployment deployment-name -n ns
  ```

  Add taint tolerance in *spec* in the yaml file:
  
  ```bash
  tolerations:
  	- key: "special"
  	operator: "Equal"
  	value: "true"
  	effect: "NoSchedule"
  ```

Then use  `kubectl describe pod` and find that default-scheduler scheduled the pod successfully:

```bash
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  46m (x18 over 132m)  default-scheduler  0/2 nodes are available: 2 Insufficient cpu, 2 Insufficient memory. preemption: 0/2 nodes are available: 2 No preemption victims found for incoming pod..
  Warning  FailedScheduling  20m                  default-scheduler  0/2 nodes are available: 2 Insufficient cpu, 2 Insufficient memory. preemption: 0/2 nodes are available: 2 No preemption victims found for incoming pod..
  Normal   Scheduled         17m                  default-scheduler  Successfully assigned vcd-ose/vcd-ose-6377fd78-c7c1-4ad8-a222-41dd23605044-5fd9957587-ldqpz to kind-worker
  Normal   Pulling           17m                  kubelet            Pulling image "***"
  Normal   Pulled            14m                  kubelet            Successfully pulled image "***" in 3m13.030336154s (3m13.03038557s including waiting)
  Normal   Created           14m                  kubelet            Created container ***
  Normal   Started           14m                  kubelet            Started container ***
  Warning  Unhealthy         10m (x22 over 13m)   kubelet            Readiness probe failed: Get "http://10.244.1.3:8080/api/v1/core": dial tcp 10.244.1.3:8080: connect: connection refused
```

## Reference

[腾讯云 - Pod一直处于 Pending状态](https://cloud.tencent.com/document/product/457/42948)

