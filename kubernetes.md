# Kubernetes Architecture

Kubernetes uses a **control plane and worker node architecture**. The control plane manages the cluster's desired state, while worker nodes run containerized application workloads.

## High-Level Architecture

```text
Users / CI-CD / kubectl
          |
          v
+-------------------------------+
|         Control Plane         |
|-------------------------------|
| kube-apiserver                |
| etcd                          |
| kube-scheduler                |
| kube-controller-manager       |
| cloud-controller-manager      |
+-------------------------------+
          |
          | Kubernetes API
          v
+-------------------------------+    +-------------------------------+
|         Worker Node 1         |    |         Worker Node 2         |
|-------------------------------|    |-------------------------------|
| kubelet                       |    | kubelet                       |
| kube-proxy                    |    | kube-proxy                    |
| Container Runtime             |    | Container Runtime             |
|                               |    |                               |
| Pod A          Pod B          |    | Pod C          Pod D          |
+-------------------------------+    +-------------------------------+
```

## 1. Control Plane

The **control plane** manages the Kubernetes cluster. It processes API requests, stores cluster state, schedules Pods, and runs control loops that continuously move the actual cluster state toward the declared desired state.

### kube-apiserver

The `kube-apiserver` exposes the Kubernetes API and acts as the front end of the control plane.

Responsibilities include:

- Receiving requests from `kubectl`, automation tools, and other components
- Authenticating and authorizing requests
- Validating Kubernetes objects
- Reading and writing cluster data through `etcd`
- Providing the communication point for cluster components

```text
kubectl -> kube-apiserver -> etcd
                    |
                    +-> scheduler
                    +-> controllers
                    +-> kubelets
```

The API server can be scaled horizontally by running multiple instances behind a load balancer.

### etcd

`etcd` is a consistent, highly available key-value store used as Kubernetes' backing store.

It stores information such as:

- Nodes
- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- Cluster configuration
- Current and desired state

> Back up `etcd` regularly because it contains the persistent state needed to recover the cluster.

### kube-scheduler

The `kube-scheduler` watches for newly created Pods that have not yet been assigned to a node. It selects a suitable worker node for each Pod.

Scheduling decisions can consider:

- Requested CPU and memory
- Available node resources
- Node selectors
- Node affinity and anti-affinity
- Pod affinity and anti-affinity
- Taints and tolerations
- Topology-spread constraints
- Persistent storage requirements
- Scheduling priorities

The scheduler selects a node, but the node's `kubelet` is responsible for starting the Pod.

### kube-controller-manager

The `kube-controller-manager` runs multiple control loops called controllers.

Important controllers include:

- **Deployment controller** — manages Deployment rollouts
- **ReplicaSet controller** — maintains the requested number of Pod replicas
- **Node controller** — monitors node health
- **Job controller** — manages workloads that run to completion
- **EndpointSlice controller** — maintains network endpoint information for Services
- **ServiceAccount controller** — manages default service accounts

Controllers compare the current cluster state with the desired state and take corrective action when the states differ.

### cloud-controller-manager

The optional `cloud-controller-manager` connects Kubernetes with a supported cloud provider.

It can manage:

- Cloud load balancers
- Node information
- Routes
- Cloud storage integration
- Node lifecycle based on cloud infrastructure

Its presence and behavior depend on the Kubernetes distribution and cloud platform.

## 2. Worker Nodes

Worker nodes provide the compute resources on which application Pods run. Every worker node normally includes a `kubelet`, a container runtime, and networking components.

### kubelet

The `kubelet` is an agent running on each worker node.

Its responsibilities include:

- Registering the node with the cluster
- Watching for Pods assigned to the node
- Asking the container runtime to create containers
- Mounting required volumes
- Running health probes
- Reporting Pod and node status to the API server

The kubelet does not normally manage containers that were created outside Kubernetes.

### Container Runtime

The container runtime runs the containers inside Pods.

Common CRI-compatible runtimes include:

- `containerd`
- `CRI-O`

Kubernetes communicates with the runtime through the **Container Runtime Interface (CRI)**.

### kube-proxy

`kube-proxy` is an optional node networking component that helps implement Kubernetes Services. It maintains network rules that direct Service traffic to the appropriate backend Pods.

Depending on the environment, service networking may be implemented using technologies such as:

- `iptables`
- `nftables`
- IP virtual server mechanisms
- An alternative networking data plane supplied by a CNI implementation

## 3. Pods

A **Pod** is the smallest deployable unit in Kubernetes. A Pod contains one or more tightly coupled containers that are scheduled together on the same node.

Containers in a Pod share:

- A network namespace
- A Pod IP address
- Port space
- Attached Kubernetes volumes
- Lifecycle and scheduling context

```text
Pod
├── Main application container
├── Optional sidecar container
└── Shared network and volumes
```

Pods are temporary and replaceable. Workload controllers usually create and manage Pods instead of users creating them directly.

## 4. Kubernetes API and Declarative State

Kubernetes uses a declarative model:

1. A user submits the desired state to the API server.
2. The API server validates and stores the object.
3. Controllers observe the new desired state.
4. The scheduler assigns unscheduled Pods to nodes.
5. The kubelet starts the required containers.
6. Controllers continuously reconcile differences between desired and actual state.

Example Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```

Apply it with:

```bash
kubectl apply -f deployment.yaml
```

Kubernetes then attempts to keep three matching Pods running.

## 5. Request Flow: Creating a Pod

When a Pod is created, the request usually follows this flow:

```text
1. kubectl sends the manifest to kube-apiserver
2. kube-apiserver authenticates, authorizes, and validates it
3. The Pod definition is persisted in etcd
4. kube-scheduler selects a worker node
5. The node's kubelet detects the assignment
6. kubelet asks the container runtime to pull the image
7. The runtime creates and starts the container
8. CNI networking assigns and configures Pod networking
9. kubelet reports Pod status through kube-apiserver
```

## 6. Cluster Networking

Kubernetes networking is designed around several important principles:

- Each Pod receives its own cluster IP address.
- Containers in the same Pod communicate through `localhost`.
- Pods should be able to communicate across nodes through the cluster network.
- Services provide stable access to changing groups of Pods.
- DNS provides name resolution for Services and, where configured, Pods.

Important networking elements include:

### CNI Plugin

A **Container Network Interface (CNI)** plugin configures Pod networking. Depending on the implementation, it may also enforce network policy and provide routing or observability capabilities.

### Service

A Service provides a stable virtual endpoint for a dynamic set of Pods selected through labels.

```text
Client -> Service -> Pod 1
                  -> Pod 2
                  -> Pod 3
```

### Ingress or Gateway

Ingress or Gateway API resources can route external HTTP and HTTPS traffic to Services. An appropriate controller must be installed for these resources to function.

### CoreDNS

CoreDNS commonly provides internal DNS-based service discovery.

Example internal name:

```text
web.default.svc.cluster.local
```

## 7. Storage Architecture

Containers and Pods are replaceable, so persistent application data should be stored outside a container's writable layer.

Important storage objects include:

- **Volume** — storage mounted into containers in a Pod
- **PersistentVolume (PV)** — storage capacity available to the cluster
- **PersistentVolumeClaim (PVC)** — a workload's request for persistent storage
- **StorageClass** — describes a storage class and enables dynamic provisioning
- **CSI driver** — integrates Kubernetes with a storage system through the Container Storage Interface

```text
Pod -> PersistentVolumeClaim -> PersistentVolume -> Storage System
```

## 8. Add-ons

Cluster add-ons extend Kubernetes functionality. Common examples include:

- DNS
- Metrics collection
- Centralized logging
- Dashboard or management interfaces
- Ingress or Gateway controllers
- Network-policy engines
- Monitoring and alerting platforms

These add-ons usually run as Kubernetes workloads inside the cluster.

## 9. High-Availability Architecture

A production cluster commonly uses:

- Multiple control-plane nodes
- Multiple API server instances behind a load balancer
- An odd-numbered, highly available `etcd` cluster
- Multiple worker nodes across failure domains
- Replicated application Pods
- Pod anti-affinity or topology-spread constraints
- PodDisruptionBudgets for controlled disruptions
- Regular `etcd` backups
- Monitoring, logging, and alerting

```text
                  Load Balancer
                        |
        +---------------+---------------+
        |               |               |
 Control Plane 1  Control Plane 2  Control Plane 3
        |               |               |
        +-------- Highly Available -----+
                    etcd cluster
                        |
        +---------------+---------------+
        |               |               |
     Worker 1        Worker 2        Worker 3
```

## 10. Important Architecture Interfaces

Kubernetes uses standardized interfaces to keep core components extensible:

- **CRI** — Container Runtime Interface
- **CNI** — Container Network Interface
- **CSI** — Container Storage Interface

These interfaces allow compatible runtime, networking, and storage implementations to integrate with Kubernetes.

## Architecture Summary

```text
Control Plane
├── kube-apiserver: exposes and processes the Kubernetes API
├── etcd: stores cluster state
├── kube-scheduler: assigns Pods to suitable nodes
├── kube-controller-manager: runs reconciliation controllers
└── cloud-controller-manager: integrates with cloud APIs

Worker Node
├── kubelet: manages assigned Pods
├── container runtime: runs containers
├── kube-proxy or alternative data plane: implements service networking
└── Pods: run application containers

Supporting Systems
├── CNI: Pod networking
├── CSI: storage integration
├── CoreDNS: service discovery
└── Ingress/Gateway controllers: inbound traffic routing
```

## References

- [Kubernetes documentation: Cluster Architecture](https://kubernetes.io/docs/concepts/architecture/)
- [Kubernetes documentation: Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
- [Kubernetes documentation: Concepts](https://kubernetes.io/docs/concepts/)


# Pods in Kubernetes

A **Pod** is the **smallest deployable unit in Kubernetes**. It contains one or more containers that run together on the same worker node. Containers within a Pod share networking, storage volumes, and lifecycle configuration.

```text
Kubernetes Cluster
└── Worker Node
    ├── Pod
    │   └── Application Container
    └── Pod
        ├── Application Container
        └── Sidecar Container
```

## Key Characteristics

- A Pod usually contains **one application container**.
- Multiple containers should be placed in the same Pod only when they are tightly coupled.
- Every Pod receives its own **cluster-internal IP address**.
- Containers inside a Pod communicate through `localhost`.
- Containers can share data through Kubernetes **Volumes**.
- All containers in a Pod are scheduled onto the **same node**.
- Pods are disposable—Kubernetes can replace a failed Pod with a new Pod that may have a different IP address.

## Basic Pod Manifest

Create a file named `pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
```

Apply the manifest:

```bash
kubectl apply -f pod.yaml
```

## Common Pod Commands

### List Pods

```bash
kubectl get pods
```

### Show Additional Pod Details

```bash
kubectl get pod nginx-pod -o wide
```

### Describe a Pod

```bash
kubectl describe pod nginx-pod
```

### View Container Logs

```bash
kubectl logs nginx-pod
```

For a multi-container Pod, specify the container:

```bash
kubectl logs nginx-pod -c nginx
```

### Execute a Command Inside a Container

```bash
kubectl exec -it nginx-pod -- /bin/sh
```

### Delete a Pod

```bash
kubectl delete pod nginx-pod
```

## Pod Lifecycle Phases

| Phase | Meaning |
|---|---|
| `Pending` | The Pod is accepted, but one or more containers have not started. |
| `Running` | The Pod is assigned to a node and at least one container is running. |
| `Succeeded` | All containers completed successfully. |
| `Failed` | At least one container terminated unsuccessfully. |
| `Unknown` | Kubernetes cannot determine the Pod's state. |

> **Note:** A Pod can be in the `Running` phase but still not be ready to receive traffic. Check readiness conditions and probes when validating application health.

## Container Types in a Pod

### Application Containers

Run the main application workload.

### Init Containers

Run before application containers start. They are commonly used for initialization tasks such as waiting for dependencies, preparing configuration, or setting permissions.

### Sidecar Containers

Run alongside the main application container and provide supporting capabilities such as logging, monitoring, proxying, or configuration synchronization.

### Ephemeral Containers

Can be added temporarily to a running Pod for debugging and troubleshooting.

## Health Probes

### Liveness Probe

Checks whether the container is healthy. Kubernetes can restart the container when the probe repeatedly fails.

### Readiness Probe

Checks whether the container is ready to receive traffic. A failed readiness probe removes the Pod from matching Service endpoints.

### Startup Probe

Checks whether a slow-starting application has finished starting. It prevents liveness and readiness probes from interfering during startup.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-probes
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 10
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 15
        periodSeconds: 20
```

## Common Pod Errors

### `CrashLoopBackOff`

The container starts and repeatedly crashes.

```bash
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>
```

Check:

- Application startup command
- Environment variables
- ConfigMaps and Secrets
- Health probes
- CPU and memory limits
- Application logs

### `ImagePullBackOff`

Kubernetes cannot download the container image.

```bash
kubectl describe pod <pod-name>
```

Check:

- Image name and tag
- Container registry availability
- Registry credentials
- `imagePullSecrets`
- Network connectivity from the node

### Pod Stuck in `Pending`

The scheduler cannot place the Pod on a node.

```bash
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

Common causes:

- Insufficient CPU or memory
- Node selector mismatch
- Affinity or anti-affinity rules
- Node taints without matching tolerations
- Unbound PersistentVolumeClaims

## Pods vs Deployments

In production, Pods are generally not created directly. Instead, use a workload controller such as:

- `Deployment` for stateless applications
- `StatefulSet` for stateful applications
- `DaemonSet` for one Pod per applicable node
- `Job` for tasks that run to completion
- `CronJob` for scheduled tasks

A controller maintains the desired number of Pods and replaces failed Pods automatically.

```text
Deployment
   ↓ manages
ReplicaSet
   ↓ creates and replaces
Pods
   ↓ contain
Containers
```

Example Deployment:

```bash
kubectl create deployment nginx --image=nginx:1.27 --replicas=3
kubectl get deployment,pods
```

## Summary

- A Pod is Kubernetes' smallest deployable unit.
- A Pod contains one or more tightly coupled containers.
- Containers in a Pod share networking and can share storage.
- Pods are temporary and replaceable.
- Use workload controllers such as Deployments instead of managing production Pods directly.

## References

- [Kubernetes documentation: Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Kubernetes documentation: Workloads](https://kubernetes.io/docs/concepts/workloads/)
