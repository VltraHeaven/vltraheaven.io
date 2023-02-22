+++
title = "Certified Kubernetes Security Specialist Exam Notes"
date = "2023-02-21"
author = "Julio Hawthorne"
banner = "img/cks.jpg"
description = "Notes for the CKS exam."
tags = ["kubernetes"]
keywords = ["devops", "education", "linux", "kubernetes", "cks"]
+++

# Table of Contents
1. [Kubernetes Secure Architecture](#kubernetes-secure-architecture)
2. [Containers Under the Hood](#containers-under-the-hood)
3. [Network Policies](#network-policies)
4. [GUI Elements](#gui-elements)

<br>

# Kubernetes Secure Architecture

Kubernetes certificates are located in `/etc/kubernetes/pki`.

The `kubelet`, `kube-controller-manager`, and `kube-scheduler` client certificates are held in kubeconfigs at `/etc/kubernetes/*.conf`.

The kubelet server certificates are held in `/var/lib/kubelet`.

# Containers Under the Hood

Applications within containers and their associated binaries operate by making syscalls to the shared kernel's syscall interface. In this way, all containers running on a node have a level of access to the host's kernel.

Vulnerabilites in the linux kernel could degrade the level of isolation between applications running within containers.

CGroups restrict a processes resource use when interacting with the host's kernel:
- RAM
- Disk
- CPU

Namespaces restrict what processes can see:
- PID - isolates processes, allows the same PID number to be used on the same host in different namespaces
- Mount - Restrict access to mounts and the root filesystem
- Network - Only allows access to certain devices, firewalls, restricts visibility of traffic from outside the namespace
- User - Isolates user ids and restricts use of the host's root user within a container/namespace 

# Network Policies

`NetworkPolicy` resources behave as Firewall rules in Kubernetes. These are implemented by the CNI and exist at the Namespace level.

They can be used to Ingress/Egress traffic for a pod, group of pods, or all pods in a namespace based on rules and conditions.

Example Network Policy:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-network-policy
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - port: 80
          protocol: TCP
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: db
        - ipBlock:
	        cidr: 10.0.0.0/24
      ports:
        - port: 3306
```
- `podSelector` is used to apply a `NetworkPolicy` to a pod or group of pods.
- `podSelector` and `namespaceSelector` are used to match pods for Ingress/Egress rules.
- An `ipBlock` can also be used to set rules for a specific ip range using CIDR notation.
- An empty rule definition (`egress: {}`, `ingress: {}`) acts as a deny-all rule for that direction of traffic flow.

Example deny-all NetworkPolicy:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
```

If a pod has more than one matching `NetworkPolicy`, the effect of all rules in each `NetworkPolicy` will be joined together and applied to the pod.

# GUI Elements

Kubernetes Dashboard Links:

- [Deploy and Access the Kubernetes Dashboard | Kubernetes](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
- [GitHub - kubernetes/dashboard: General-purpose web UI for Kubernetes clusters](https://github.com/kubernetes/dashboard)
- [Kubernetes Dashboard Arguments](https://github.com/kubernetes/dashboard/blob/master/docs/common/dashboard-arguments.md)

The Kubernetes Dashboard does not allow concurrent HTTP and HTTPS access.

RBAC for the `kubernetes-dashboard` service account can be adjusted with `ClusterRole`/`Role` and `ClusterRoleBinding`/`RoleBinding` resources to manage what is visible in the Kubernetes Dashboard.

Interesting Kubernetes Dashboard Arguments:

- `--authentication-mode`: Specify basic auth, RBAC, and Token based auth
- `--enable-skip-login`: Skips login when accessing the Dashboard
- `--auto-generate-certificates`: Dashboard automatically generates TLS certificates for HTTPS access
- `--tls-cert-file`/`--tls-key-file`: Specify custon TLS cert/key for HTTPS access
- `--insecure-port`: Specify a port to be used for unencrypted HTTP access