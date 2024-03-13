---
title: "Preparation for CKA"
date: 2024-03-13T17:45:35+08:00
draft: false
tags: ["k8s", "cka"]
categories: ["k8s"]
author: "大白猫"
---

## Official Website

* [K8s doc](https://kubernetes.io/docs/home/)
* [Training portal](https://trainingportal.linuxfoundation.org/learn/dashboard)

## Prepare

### Resources in Github

- [Kubernetes Certified Administration](https://github.com/walidshaari/Kubernetes-Certified-Administrator) - Some exercises based on exam objectives. Has some useful tips and links.
- [K8s Practice Training](https://github.com/StenlyTU/K8s-training-official) - A basic k8s exercise.
- [Awesome Kubernetes](https://github.com/ramitsurana/awesome-kubernetes) - Introduction and blogs about k8s.

### Online Practice

https://killercoda.com/killer-shell-cka - Free

https://killer.sh - two sessions, 36h free for each

### Related Blogs (In Chinese)

* [CKA 考試全攻略流程](https://medium.com/@app0/cka-%E8%80%83%E8%A9%A6%E5%85%A8%E6%94%BB%E7%95%A5%E6%B5%81%E7%A8%8B-3a28d1b73eea)
* [Kubernetes CKA 证书备考笔记](https://atbug.com/notes-for-cka-preparation/)

## Practice

### 1 RBAC

Create a cluster role *deployment-clusterrole* which can create Deployment, StatefulSet, DaemonSet.

Create a service account *cicd-token* in namespace *app-team1*.

Bind cluster role *deployment-clusterrole* to the service account *cicd-token*.

### 2 Scale Deployment

Extend deployment *presentation* to 4 pods.

### 3 NetworkPolicy

Create a new NetworkPolicy *allow-port-from-namespace* in namespace *my-app*. 

Ensure the new NetworkPolicy allows the pods in namespace *echo* to access port *9000* of pods in namespace *my-app*.

Further ensure the new NetworkPolicy:

* Forbidden the access from the pods which doesn't listen to port 9000.
* Forbidden the access from the pods which aren't in namespace *echo*.

### 4 Expose Service

Re-configure the current deployment *front-end* to add a http port for container *nginx*: 80/tcp.

Create a service *front-end-svc* to expose the port.

Configure this service to expose the pods by their NodePort.

### 5 Create Ingress

Create a nginx Ingress:

- Name: ping
- Namespace: ing-internal
- Use port *5678* and path */hello* to expose service *hello*.

Can use this command to check the availability of service *hello*: `curl -kL <INTERNAL_IP>/hello`

### 6 Get Pod Using Highest CPU

Find the running pods using much CPU resources by the pod label *name=cpu-loader*.

Write the name of the pod using highest CPU to file */opt/KUTR000401/KUTR00401.txt* (existed).

### 7 Schedule Pod to Specified Node

Schedule the pod:

- Name: nginx-kusc00401
- Image: nginx
- Node selector: disk=ssd

### 8 Check Available Nodes

Check how many nodes is Ready (except the nodes with *Taint:NoSchedule*).

Write the number of ready nodes to */opt/KUSC00402/kusc00402.txt*.

### 9 Create Pod with Multiple Container

Create a pod:

- Name: kucc8
- App containers: 2
- Container name / Image: nginx, memcached

### 10 Create PV

Create a persistent volume:

- Name: app-config
- Capacity: 1Gi
- Access mode: ReadWriteMany
- Volume location: hostPath - */srv/app-config*

### 11 Create PVC

Create a new PersistentVolumeClaim:

- Name: pv-volume
- Storage class: csi-hostpath-sc
- Capacity: 10Mi

Create a new pod to mount volume:

- Name: web-server
- Image: nginx:1.16
- Mount path: /usr/share/nginx/html

Configure this pod to have ReadWriteOnce permission on the volume.

At last, use `kubectl edit` or `kubectl patch` to extend the capacity of PVC to 70Mi, and record this change.

### 12 View Pod Logs

Monitor log of pod *foo* and:

- Extract the log corresponding to error *RLIMIT_NOFILE*
- Write these log to */opt/KUTR00101/foo*

### 13 Use Sidecar to Proxy Container Logs

Use busybox image to add a sidecar container named *sidecar* to the existing pod *11-factor-app*. 

The new sidecar container should run the following command: `/bin/sh -c tail -n+1 -f /var/log/11-factor-app.log`. 

Use a Volume mounted at */var/log* to make the log file *11-factor-app.log* available to the sidecar container.

Do not modify the specifications of existing containers other than adding the required volume mount.

### 14 Upgrade Cluster

The existing Kubernetes cluster is running version 1.29.0. **Only** upgrade all Kubernetes control plane and node components on the **master node** to version 1.29.1. 

Ensure to drain the master node before the upgrade and uncordon it after the upgrade.

You can use the following command to SSH into the master node: `ssh master01`

You can use the following command to obtain higher privileges on this master node: `sudo -i`

Additionally, upgrade kubelet and kubectl on the master node. Please do not upgrade worker nodes, etcd, container manager, CNI plugins, DNS service, or any other plugins.

### 15 Backup and restore etcd

You must execute the required `etcdctl` command from the host *master01*.

First, create a snapshot of the existing etcd instance running on https://127.0.0.1:2379 and save the snapshot to */var/lib/backup/etcd-snapshot.db*.

The following TLS certificate and key are provided to connect to the server via `etcdctl`: 

- CA certificate: /opt/KUIN00601/ca.crt 
- Client certificate: /opt/KUIN00601/etcd-client.crt 
- Client key: /opt/KUIN00601/etcd-client.key

Creating a snapshot for the given instance is expected to complete within a few seconds. If the operation seems to hang, there might be an issue with the command. Use CTRL + C to cancel the operation and then retry. 

Then, restore from the previous backup snapshot located at */data/backup/etcd-snapshot-previous.db*.

### 16 Troubleshoot Faulty Node

The Kubernetes worker node *node02* is in NotReady. 

Investigate why this is happening and take action to return this node to the Ready state, ensuring that the changes are permanent.

### 17 Node Maintenance

Make the node *node02* unavailable and reschedule all pods running on this node.

## Tips

* Use `alias` to simplify the command line

  ```sh
  alias kc='kubectl'
  ```

* Use `:set paste` before edit yaml file to guarantee format

## Timeline

Buy: Cyber Monday 2023 - about 1800 Chinese yuan for CKA and CKS

Prepare: 2024.1-2024.2

Exam: 2024.3.9 1:00 AM
