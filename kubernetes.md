Pods in Kubernetes

A Pod is the smallest deployable unit in Kubernetes. It contains one or more containers that run together on the same worker node. Containers within a Pod share networking, storage volumes, and lifecycle configuration.

Kubernetes Cluster
└── Worker Node
    ├── Pod
    │   └── Application Container
    └── Pod
        ├── Application Container
        └── Sidecar Container

Key characteristics
A Pod usually contains one application container.
Multiple containers should be placed in the same Pod only when they are tightly coupled.
Every Pod receives its own cluster-internal IP address.
Containers inside the Pod communicate through localhost.
Containers can share data through Kubernetes Volumes.
All containers in a Pod are scheduled onto the same node.
Pods are disposable—Kubernetes can replace a failed Pod with a new Pod that has a different IP address.
Basic Pod manifest
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


Create the Pod:

kubectl apply -f pod.yaml


Check it:

kubectl get pods
kubectl get pod nginx-pod -o wide
kubectl describe pod nginx-pod


Access container logs:

kubectl logs nginx-pod


Execute a command inside the container:

kubectl exec -it nginx-pod -- /bin/sh


Delete the Pod:

kubectl delete pod nginx-pod

Pod lifecycle phases
Phase	MeaningPending	Pod is accepted, but one or more containers have not started
Running	Pod is assigned to a node and at least one container is running
Succeeded	All containers completed successfully
Failed	At least one container terminated unsuccessfully
Unknown	Kubernetes cannot determine the Pod’s state

A Pod can be Running but still not be ready to receive traffic, so readiness conditions and probes are more meaningful than phase alone when checking application health.

Common Pod errors
CrashLoopBackOff

The container starts and repeatedly crashes.

kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>


Check the application command, environment variables, configuration, secrets, probes, and resource limits.

ImagePullBackOff

Kubernetes cannot download the container image.

kubectl describe pod <pod-name>


Check:

Image name and tag
Registry availability
imagePullSecrets
Registry authentication
Pod stuck in Pending

The scheduler cannot place the Pod on a node.

kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp


Common causes include insufficient CPU or memory, node selectors, affinity rules, taints, and unbound storage claims.

Pods versus Deployments

In production, you normally should not create Pods directly. Instead, use a controller such as a Deployment, StatefulSet, DaemonSet, or Job. Controllers maintain the desired number of Pods and replace failed Pods automatically.

For example:

kubectl create deployment nginx --image=nginx:1.27 --replicas=3
kubectl get deployment,pods


A useful mental model is:

Deployment
   ↓ manages
ReplicaSet
   ↓ creates and replaces
Pods
   ↓ contain
Containers
