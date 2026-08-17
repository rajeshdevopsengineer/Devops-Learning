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
