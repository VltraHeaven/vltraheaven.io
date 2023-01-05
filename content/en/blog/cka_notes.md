+++
title = "Kubernetes Exam Notes"
date = "2020-12-04"
author = "Julio Hawthorne"
banner = "img/terminal.gif"
description = "Notes for the CKA, CKAD, and CKS exams."
tags = ["kubernetes"]
keywords = ["devops", "education", "linux", "kubernetes", "cka", "ckad", "cks"]
+++

## Certified Kubernetes Application Developer

---

### Kubernetes Architecture

Pod: One or more containers that share a single IP address, storage and a namespace. Usually one container runs the primary application with one or more secondary containers providing supporting services. The smallest unit used in a cluster to deploy an application. Pods are typically designed to provide a single process-per-container. To support this architecture, it may be necessary to run logging, a proxy or a special adapter, usually handled by other containers in the same pod. Containers can use IPC, the loopback interface, or a shared filesystem to communicate. 

Sidecar: A container dedicated for performing a helper task, like logging or acting as a proxy, for a pod.

InitContainers: Since containers within a pod are created in parallel, this mechanism can help to ensure that dependent containers are started only after the containers hosting their dependencies.

ReplicaSet: A controller that deploys and restarts containers until the requested number of containers are running.

Depoloyment: Ensures that resources are available (IP addressing, storage, etc.) and then creates a ReplicaSet.

Job: Handles running a single one-shot task

CronJob: Handles running a recurring task.

DaemonSet: Ensures that a single pod is deployed on each node in a cluster/namespace. Often used for logging and metrics pods.

StatefulSet: Used to deploy pods in a specified order so that following pods are only deployed if previous pods report a ready status. Useful when one pod's service has a dependency on a service running in another pod.

NodeAffinity: A property of pods that attracts them to a set of nodes.

Taints: A property of pods that repels them from deploying on a set of nodes.

Tolerations: A property of pods that allows, but does not require, them to schedule onto a set of nodes that have a matching taint applied.

Predicates: A Method the kube-scheduler uses to remove worker nodes from the selection of available worker nodes.

Priorities: A series of characteristics the kube-scheduler uses to choose the best worker node(s) to host a pod.

> Additional annotations can be made to the metadata that can be utilized by third-party agents or tools

Multi-tenancy: Allowing several different users or groups to share a cluster.

Namespace: A segregation of resources in a cluster upon which quotas and permissions can be applied. Objects can be created within a namespace or be scoped for an entire cluster. Users can be limited by the object verbs of a namespace and the `LimitRange` admission controller can be used to control resource usage. Two objects cannot share the same `Name:` in the same Namespace.

Context: A combination of user, cluster name, and namespace to allow easy switching between combinations of permissions and restrictions. This info is referenced in the `.kube/config` file.

Resource Limits: Method of restricting the amount of resources for a pod, or to request a minimum amount of resources reserved. Can also be set across entire namespaces, having priority over the settings in the PodSpec.

Pod Security Policies: Limits the ability for pods to escalate privileges or modify the nodes they are scheduled on. May prevent some pods from operating properly.

Network Policies: Allows a firewall within the cluster. Ingress/Egress traffic can be limited according to namespaces and labels as well.

Master Node: Runs various server and manager processes for the cluster.

> kube-apiserver: The server that acts as the central nervous system of a cluster and that handles all internal and external calls through a cluster. All actions are accepted and validated by this agent, and is the only agent with permissions to connect to etcd. Each API call goes through three steps: authentication, authorization and several admission contrillers.

> kube-scheduler: Algorithmically determines which node will host a pod. Views available resources to bind, and then attempts to deploy a pod based on availability and success. Custom schedulers can be used instead and pods can be bound to specific nodes. First, the quota restrictions are evaluated to determine of a pod can be deployed on a particular node. Next, the taints, tolerations and labels are evaluated to determine the appropriate node placement of a pod.

> etcd: A database that stores the state of a cluster, networking attributes and other persistent information in a b+tree key-value store. Values are always appended to the end of an entry and prevous entries are marked for future deletion by a compaction process. Requests are routed through through the kube-apiserver to etcd synchronously. An initial request would update etd and a second request would trigger the kube-apiserver to respond with a 409 error due to not having the same version number. A client would need to expect the possibility of this behavior and act upon the denial to update.

> kube-controller-manager: The brain of a kubernetes cluster. A core control loop daemon that interacts with the kube-apiserver to determine the state of the cluster. The manager will contact the necessary controller to correct any discrepancies in the state of the cluster.

> cloud-controller-manager: Interacts with agents outside of the cloud. Handles tasks once handled by the kube-controller-manager and allows faster changes without altering the core kubernetes control process. Each kubelet must use the `--cloud-provider-external` settings passed to the binary.

Worker Nodes: Worker nodes run the kubelet, kube-proxy, and a container engine (docker, gvisor, CRI-O). Other management daemons are deployed as well to monitor agents or provide additional services.

> kubelet: The primary node agent that runs on each node. Takes a set of PodSpecs and interacts with the underlying container engine to ensure that the containers described are running and healthy.

> PodSpec: a JSON or YAML file that describes a pod. Provides information to the kubelet on how to configure a pod.

> kube-proxy: Manages the network connectivity of containers using iptables entries. It's userspace mode monitors services and endpoints, proxying traffic through a random dynamic port. Ipvs can be enabled with the expectation that it will become the de-facto network management tool in the future.

> Fluentd: Provides a unified logging mechanism for a cluster, providing filters, buffers and routing for messages. Provided by the CNCF and not included by default.

> Prometheus: A time-based metrics database used to gather metrics from nodes and applications. Not included by default.

Services: A microservice that handles a particular bit of traffic, such as a single NodePort or a LoadBalancer to distribute inbound requests among many Pods. Services also handle access policies for inbound requests for resource control and security. Uses selectors to identify which obnjects to connect, as does the `kubectl` cli tool.

> equality-based (selector): Allows services to identify objects to connect by keys and values. Uses the operators `=`, `==`, and `!=`. If multiple values or keys are used, all must be included for a match.

> set-based (selector): Allows services to identify which objects to connect based of a set of values. The use of `status notin (dev, test, maint)` would select resources with the key of `status` which did not have a value of `dev, test,` nor `maint`. 

Controllers: Also known as watch-loops or operators. They query the current spec of a pod, compare it against the spec and execute code dependent on the differences. Several contrillers ship with kubernetes, and it's possible to create one's own. As long as the deltas are not of the type *Deleted*, the logic of the controller is used to create or modify some object until it matches the spec. The `endpoints`, `namespace`, and `serviuceaccunts` controllers manage the appropriately named resources for Pods.

Pause Container: A container initalized within a pod before all other containers that is responsible for obtaining an IP address for the pod, then sharing it's namespaces with the pods defined in the PodSpec. Can be inspected using the `ps` command. e.g. `sudo docker ps | grep -i pause`. 

[Container Network Interface (CNI) Specification](https://github.com/containernetworking/cni): A specification with associated libraries for writing plugins used to configure container networking and remove allocated resources when containers are deleted. It provides a common interface between various networking solutions and container runtimes. Since the CNI spec is language-agnostic, there are several plugins for a range of network solutions.

Software Defined Network Overlays:
> [Weave](https://weave.works)
> [Flannel](https://github.com/coreos/flannel)
> [Calico](https://projectcalico.org)
> [Romana](https://romana.io)
