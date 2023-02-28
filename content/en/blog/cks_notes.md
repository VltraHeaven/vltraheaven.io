+++
title = "Certified Kubernetes Security Specialist Exam Notes"
date = "2023-02-21"
author = "Julio Hawthorne"
banner = "img/cks.jpg"
description = "Notes for the CKS exam."
tags = ["Kubernetes"]
keywords = ["DevOps", "education", "Linux", "Kubernetes", "cks"]
+++

# Table of Contents
1. [Kubernetes Secure Architecture](https://vltraheaven.io/blog/cks_notes/#kubernetes-secure-architecture)
2. [Containers Under the Hood](https://vltraheaven.io/blog/cks_notes/#containers-under-the-hood)
3. [Network Policies](https://vltraheaven.io/blog/cks_notes/#network-policies)
4. [GUI Elements](https://vltraheaven.io/blog/cks_notes/#gui-elements)
5. [Secure Ingress](https://vltraheaven.io/blog/cks_notes/#secure-ingress)
6. [Node Metadata Protection](https://vltraheaven.io/blog/cks_notes/#node-metadata-protection)
7. [CIS Benchmarks](https://vltraheaven.io/blog/cks_notes/#cis-benchmarks)
8. [Verifying Platform Binaries](https://vltraheaven.io/blog/cks_notes/#verifying-platform-binaries)
9. [RBAC](https://vltraheaven.io/blog/cks_notes/#rbac)
<br>

# Kubernetes Secure Architecture

Kubernetes certificates are located in `/etc/kubernetes/pki`.

The `kubelet`, `kube-controller-manager`, and `kube-scheduler` client certificates are held in kubeconfigs at `/etc/kubernetes/*.conf`.

The kubelet server certificates are held in `/var/lib/kubelet`.

# Containers Under the Hood

Applications within containers and their associated binaries operate by making syscalls to the shared kernel's syscall interface. In this way, all containers running on a node have a level of access to the host's kernel.

Vulnerabilities in the Linux kernel could degrade the level of isolation between applications running within containers.

CGroups restrict a processes resource use when interacting with the host's kernel:
- RAM
- Disk
- CPU

Namespaces restrict what processes can see:
- PID - isolates processes, and allows the same PID number to be used on the same host in different namespaces
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
- [GitHub - Kubernetes/dashboard: General-purpose web UI for Kubernetes clusters](https://github.com/kubernetes/dashboard)
- [Kubernetes Dashboard Arguments](https://github.com/kubernetes/dashboard/blob/master/docs/common/dashboard-arguments.md)

The Kubernetes Dashboard does not allow concurrent HTTP and HTTPS access.

RBAC for the `kubernetes-dashboard` service account can be adjusted with `ClusterRole`/`Role` and `ClusterRoleBinding`/`RoleBinding` resources to manage what is visible in the Kubernetes Dashboard.

Interesting Kubernetes Dashboard Arguments:

- `--authentication-mode`: Specify basic auth, RBAC, and Token-based auth
- `--enable-skip-login`: Skips login when accessing the Dashboard
- `--auto-generate-certificates`: Dashboard automatically generates TLS certificates for HTTPS access
- `--tls-cert-file`/`--tls-key-file`: Specify custom TLS cert/key for HTTPS access
- `--insecure-port`: Specify a port to be used for unencrypted HTTP access

# Secure Ingress

An `Ingress` acts as a configuration wrapper that provides instructions to an ingress controller on how to proxy traffic to a `Service`.

![Ingress Architecture Diagram](img/ingress_architecture.png)

A `ClusterIP` service always routes traffic to Pods.

A `NodePort` service works similarly to a `ClusterIP` service, with the additional behavior of opening a listening port on all Nodes in the cluster.

A `LoadBalancer` service behaves almost identically to a `NodePort` service (creates a Cluster IP address and opens a listening port on all nodes), with the additional behavior of creating a LoadBalancer in the cluster's Cloud Environment that routes traffic to the listening port on cluster's nodes.

The ingress controller's `NodePort` or `LoadBalancer` service will receive traffic, and the ingress controller will proxy traffic to the `ClusterIP` service of the targeted `Pod`. 

## Example Ingress Configuration:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```

## Example Ingress Configuration with Wildcard Host:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - pathType: Prefix
        path: "/bar"
        backend:
          service:
            name: service1
            port:
              number: 80
  - host: "*.foo.com"
    http:
      paths:
      - pathType: Prefix
        path: "/foo"
        backend:
          service:
            name: service2
            port:
              number: 80
```

An ingress controller uses a Generic Default Certificate to secure HTTPS traffic. It's best practice to secure HTTPS traffic using a custom TLS certificate.

## Generate a Self-Signed Certificate
```shell
$ openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

## Create a TLS `Secret`

Create a file named `testsecret-tls.yaml` with the following contents:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls
```

Then apply the configuration using `kubectl apply -f ./testsecret-tls.yaml`. This can also be done imperatively through `kubectl` using the command:
```shell
$ kubectl create secret tls testsecret-tls --cert=./cert.pem --key=./key.pem --namespace default
```

## Create an Ingress using the TLS Secret
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
      - https-example.foo.com
    secretName: testsecret-tls
  rules:
  - host: https-example.foo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
```

## Locally testing the Ingress configuration
```shell
$ curl https://https-example.foo.com -kv --resolve=https-example.foo.com:x.x.x.x
```

- Replace `x.x.x.x` with the IP Address of a worker node in the cluster
- The Ingress `NodePort` service's port number may need to be appended to the end of `https-example.foo.com` if the ingress was not deployed in the `kube-system` namespace.

```shell
$ curl https://https-example.foo.com:31407 -kv --resolve=https-example.foo.com:31407:x.x.x.x
```

# Node Metadata Protection

A managed Metadata API is expected to be present in Cloud Environments. This Metadata API will be accessible to the Virtual Machines running in the Cloud environment and can expose sensitive data. Commonly, any process that can communicate with the Metadata API server can access all values inside the Metadata server by default.

## Cloud Infrastructure Remediation Tips
- Ensure that the cloud-instance-account is assigned "least-privileges"
- Each cloud provider has a set of recommendations to follow
- Not in the hands of Kubernetes.

On the Kubernetes side of things, A `NetworkPolicy` can restrict pods' access to a Metadata API Server. 

## Restricting access to GCP's Metadata API Server using `NetworkPolicy`

`cloud-metadata-deny.yaml`
```
# all pods in namespace cannot access metadata endpoint
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        CIDR: 0.0.0.0/0
        except:
        - 169.254.169.254/32
```

`cloud-metadata-allow.yaml`
```
# only pods with label are allowed to access metadata endpoint
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-allow
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: metadata-accessor
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 169.254.169.254/32
```

Using the manifests above, any pods with the label `role: metadata-accessor` will be allowed to access the Metadata API Server.

# CIS Benchmarks

CIS Benchmarks provide default Kubernetes security rules. Instructions for applying CIS Benchmark rules are given using the default configuration file locations of `kubeadm`.

[CIS Kubernetes Benchmark v1.6.0](img/CIS_Kubernetes_Benchmark_v1.6.0.pdf)
<embed src="img/CIS_Kubernetes_Benchmark_v1.6.0.pdf" type="application/pdf" height="100%" width="100%">

[`kube-bench`](https://github.com/aquasecurity/kube-bench) checks whether Kubernetes is deployed according to security best practices as defined in the CIS Kubernetes Benchmark

- [Running kube-bench](https://github.com/aquasecurity/kube-bench/blob/main/docs/running.md#running-kube-bench)

## Running `kube-bench` inside a container
```shell
$ docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t docker.io/aquasec/kube-bench:latest --version 1.18
```

## Running `kube-bench` as a `Job` inside a Kubernetes cluster
```
$ kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml

$ kubectl get pods
NAME                      READY   STATUS              RESTARTS   AGE
kube-bench-j76s9   0/1     ContainerCreating   0          3s

# Wait for a few seconds for the job to complete
$ kubectl get pods
NAME                      READY   STATUS      RESTARTS   AGE
kube-bench-j76s9   0/1     Completed   0          11s

# The results are held in the pod's logs
kubectl logs kube-bench-j76s9
[INFO] 1 Master Node Security Configuration
[INFO] 1.1 API Server
```

# Verifying Platform Binaries

Binary validation is performed by comparing the current hash value of a file to the hash value provided by a trusted source. The hash value of a file is obtained using the `shasum` tool (`shasum -a 512 /usr/local/bin/kube-apiserver`). Binaries running within containers can also be performed using the contents of the container's process directory in `/proc`

```
# Grab the PID of the kube-apiserver container
$ ps aux | grep kube-apiserver
root        1291  0.0  0.0  48760  3720 ?        S<   Jan03   0:01 kube-apiserver...

# Find the location of the kube-apiserver binary running within the container
$ find /proc/1291/root/ -name kube-apiserver
/proc/1291/root/usr/local/bin/kube-apiserver

# Generate the hash of the kube-apiserver binary
shasum -a 512 /proc/1291/root/usr/local/bin/kube-apiserver
bc8ad13df7a9275cf6ecec93a9e0b390898f41242999d5e32a33071ffb0205b67755f54f0a828ccd09bb9ac43a6ed9d9b1eb7e36dc8dceed5befca7cc773ede0  /proc/1291/root/usr/local/bin/kube-apiserver
```

# RBAC

Role-based Access Control (RBAC) regulates access to computer or network resources based on the roles of individual users within an organization. Kubernetes has RBAC enabled by default and can be enabled manually using the `--authorization-mode=RBAC` argument for the `kube-apiserver` component. In Kubernetes, RBAC restricts the access of `Users` and `ServiceAccounts` to Kubernetes resources. Access is granted by specifying what is allowed. All other resources are denied by default.

`Role` and `ClusterRole` specify a set of permissions. `RoleBinding` and `ClusterRoleBinding` declare who gets a set of permissions specified within a `Role`/`ClusterRole`. `Role` and `RoleBinding` work at the `Namespace` level, while `ClusterRole` and `ClusterRoleBinding` work globally.

## Create a `Role` and `RoleBinding`

- Create a `Role` named "pod-reader" in the default `Namespace`

Declarative:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Imperative:
```shell
$ kubectl create role -n default pod-reader --verb=get --verb=watch --verb=list --resource=pods`
```

- Grant the `User` "jane" the permissions specified in the "pod-reader" `Role` using a `RoleBinding` named "read-pods"

Declarative:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

Imperative:
```shell
$ kubectl create rolebinding -n default read-pods --role=pod=reader --user=jane
```

## Create a `ClusterRole` and `ClusterRoleBinding`

- Create a `ClusterRole` named "secret-reader"

Declarative:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]
```

Imperative:
```shell
$ kubectl create clusterrole secret-reader --verb=get --verb=watch --verb=list --resource=secrets
```

- Grant the "manager" `Group`  the permissions specified in the "secret-reader" `ClusterRole`

Declarative:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

Imperative:
```shell
$ kubectl create clusterrolebinding read-secrets-global --clusterrole=secret-reader --group=manager
```

A `RoleBinding` can also reference a `ClusterRole` to grant the permissions defined in that `ClusterRole` to resources inside the `RoleBinding`'s namespace. This kind of reference lets you define common roles across your cluster, then reuse them within multiple namespaces. Permissions are additive for multiple `Role`/`RoleBinding` combinations, so binding a stricter `Role` to a `User` or `ServiceAccount` will not remove the permissions granted in a previously bound `Role` or `ClusterRole`. However, a `Role` cannot be bound to an account using a `ClusterRoleBinding`. RBAC permissions can be tested using the `kubectl auth can-i <verb> <resource> --as <User/Group/ServiceAccount>` command. 

You can _aggregate_ several ClusterRoles into one combined ClusterRole. A controller, running as part of the cluster control plane, watches for ClusterRole objects with an `aggregationRule` set. The `aggregationRule` defines a label [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) that the controller uses to match other ClusterRole objects that should be combined into the `rules` field of this one - [Using RBAC Authorization | Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#aggregated-clusterroles).

Here is an example aggregated ClusterRole:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-monitoring: "true"
rules: [] # The control plane automatically fills in the rules
```

If you create a new ClusterRole that matches the label selector of an existing aggregated ClusterRole, that change triggers adding the new rules into the aggregated ClusterRole. Here is an example that adds rules to the "monitoring" ClusterRole, by creating another ClusterRole labeled `rbac.example.com/aggregate-to-monitoring: true`.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-endpoints
  labels:
    rbac.example.com/aggregate-to-monitoring: "true"
# When you create the "monitoring-endpoints" ClusterRole,
# the rules below will be added to the "monitoring" ClusterRole.
rules:
- apiGroups: [""]
  resources: ["services", "endpointslices", "pods"]
  verbs: ["get", "list", "watch"]
```

Additionally, by creating a `ClusterRoleBinding` that references the "monitoring" `ClusterRole`, the "monitoring-sa" `ServiceAccount` will inherit the aggregated permissions of any `ClusterRole` labeled `rbac.example.com/aggregate-to-monitoring: true`.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-global
subjects:
- kind: ServiceAccount
  name: monitoring-sa
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: monitoring
  apiGroup: rbac.authorization.k8s.io
```

In Kubernetes a `ServiceAccount` is used by machines or pods to access the Kubernetes API. There is no "User" resource. It is assumed that a cluster-independent service manages normal users in the following ways:

-   an administrator distributing private keys
-   a user store like Keystone or Google Accounts
-   a file with a list of usernames and passwords

In this regard, _Kubernetes does not have objects which represent normal user accounts._ Normal users cannot be added to a cluster through an API call. Even though a normal user cannot be added via an API call, any user that presents a valid certificate signed by the cluster's certificate authority (CA) is considered authenticated. In this configuration, Kubernetes determines the username from the common name field in the 'subject' of the cert (e.g., "/CN=bob"). From there, the role based access control (RBAC) sub-system would determine whether the user is authorized to perform a specific operation on a resource - [Authenticating | Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#users-in-kubernetes).

## User Certificate Generation Process

![User Certificate Generation Process](img/user-certificate-creation.png)

- Use `openssl` to create a key and Certificate Signing Request.
```
$ openssl genrsa -out bob.key 2048

# The "Common Name" should match the name of the desired user in cluster (bob)
$ openssl req -new -key bob.key -out bob.csr
```
- Get the contents of `bob.csr` as a base64-encoded string
```shell
$ cat bob.csr | base64 -w 0
```
- Include the generated Certificate Signing Request in a new `CertificateSigningRequest` resource.
```yaml
# Save this as a file named bob-csr.yaml
kind: CertificateSigningRequest
metadata:
  name: bob
spec:
  groups:
  - system:authenticated
  request: <base64-encoded contents of bob.csr>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```
- The `kube-apiserver` will update the `CertificateSigningRequest` with a certificate signed with the cluster's CA.
```shell
$ kubectl create -f bob-csr.yaml
```
- Download the certificate and use in a kubeconfig for authentication.
```shell
# The condition of the CertificateSigningRequest shoud be "Approved"
$ kubectl get csr bob
# Look under the "status.certificate" field to retrieve the signed certificate
$ kubectl get csr bob -oyaml
# The retrieved certificate must be Base64 decoded before it can be used
$ echo <base64-encoded certificate> | base64 -d > bob.crt
# Add the Certificate and Key to your kubeconfig
$ kubectl config set-credential bob --client-key=bob.key --client-certificate=bob.crt --embed-certs
# Retrieve the name of the cluster from the kubeconfig
$ kubectl config view
# Set the context for the new user to the existing cluster 
$ kubectl config set-context bob --user=bob --cluster=<cluster-name>
```

There is no way to invalidate a certificate. Follow the remediation steps if a certificate is leaked:
- Remove all access from the User/Certificate using RBAC. In this case, the username cannot be used until the certificate expires.
- Create a new CA and re-issue all certificates

`ServiceAccount` is a namespaced resource that is managed by the Kubernetes API. There is a "default" `ServiceAccount` in every namespace that is used by pods to communicate with the Kubernetes API. Each `ServiceAccount` has a token `Secret` that it uses for API authentication. 

New temporary tokens can be generated using `kubectl create token <ServiceAccount name>`. The generated token aligns with the JWT standard and can be decoded with any JWT decoder - [JSON Web Token Decoder](https://jwt.io/). All pods that use a `ServiceAccount` will also have access to any of it's tokens. Because `ServiceAccount` tokens can also be stored in Secret API objects, any user with write access to Secrets can request a token, and any user with read access to those Secrets can authenticate as the service account. Be cautious when granting permissions to service accounts and read or write capabilities for Secrets.

## Specifying a `ServiceAccount` in a Deployment Manifest

```yaml
apiVersion: apps/v1 # this apiVersion is relevant as of Kubernetes 1.9
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: default
spec:
  replicas: 3
  template:
    metadata:
    # ...
    spec:
      serviceAccountName: bob-the-bot
      containers:
      - name: nginx
        image: nginx:1.14.2
```

The `ServiceAccount` details and token can be found at the `/run/secrets/kubernetes.io/serviceaccount` path from within a running container. The command `cat /proc/mounts | grep -i serviceaccount` will also display the path of the mounted `ServiceAccount` details when executed from within a running container. 

## Calling Kubernetes API from within a container using `curl` and the `ServiceAccount` token

```shell
$ curl -k https://kubernetes.default.svc -H "Authorization: Bearer $(cat /run/secrets/kubernetes.io/serviceaccount/token)"
```

In cases when a Pod does not need to communicate with the Kubernetes API, automounting of the `ServiceAccount` token can be disabled. "If both the ServiceAccount and the Pod's `.spec` specify a value for `automountServiceAccountToken`, the Pod spec takes precedence" - [Configure Service Accounts for Pods | Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#opt-out-of-api-credential-automounting).

## Disabling token automount in a `ServiceAccount` manifest 

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
automountServiceAccountToken: false
...
```

## Disabling token automount in a `Pod` manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: build-robot
  automountServiceAccountToken: false
  ...
```
