# Docker vs Kubernetes: Developer Control

## Docker

```text
DEVELOPER PC
    |
    | SSH
    v
SERVER
    |
  Docker
    |
    +-- Service A
    +-- Service B
```

Example:

```bash
ssh user@server1
docker ps
docker stop service-a
docker pull image:2.0
docker run ...
```

SSH gives the developer access to the server's terminal.

---

## Kubernetes

```text
DEVELOPER PC
    |
  kubectl
    |
    | Kubernetes API
    v
+----------------------+
|   CONTROL PLANE      |
| API Server           |
| Scheduler            |
| Controllers          |
+----------+-----------+
           |
     +-----+-----+-----+
     v           v     v
  Node 1      Node 2  Node 3
  kubelet     kubelet kubelet
     |           |      |
    Pods        Pods   Pods
```

The developer normally does not SSH into every node just to manage applications.

Instead:

```bash
kubectl apply -f deployment.yaml
kubectl get pods
kubectl scale deployment service-a --replicas=3
```

Kubernetes communicates with the nodes through its cluster components.

---

## Container Registry

The normal production flow includes a registry:

```text
DEVELOPER PC
    |
    | docker build
    v
  IMAGE
    |
    | docker push
    v
+----------------------+
| CONTAINER REGISTRY   |
| service-a:1.0        |
| service-a:2.0        |
+----------+-----------+
           |
           | Kubernetes pulls image
           v
      KUBERNETES
           |
     +-----+-----+
     v     v     v
   Node  Node  Node
    1     2     3
```

---

## What Is Installed Where?

### Developer PC

```text
kubectl
Docker (if building images)
Git
IDE
```

### Kubernetes Control Plane

```text
Kubernetes API Server
Scheduler
Controllers
```

### Each Worker Node

```text
kubelet
container runtime
Pods
```

The container runtime can be `containerd` or another Kubernetes-supported runtime. It does not have to be Docker.

---

## Complete Developer Workflow

Suppose Service A changes from version 1.0 to 2.0.

### 1. Build Java application

```bash
./gradlew build
```

Produces:

```text
service-a.jar
```

### 2. Build Docker image

```bash
docker build -t service-a:2.0 .
```

### 3. Push image

```bash
docker push registry.company.com/service-a:2.0
```

Now the registry contains:

```text
service-a:2.0
```

### 4. Tell Kubernetes to use the new image

```bash
kubectl set image deployment/service-a   service-a=registry.company.com/service-a:2.0
```

Kubernetes then manages the rollout.

```text
                 kubectl
                    |
                    v
             Kubernetes API
                    |
                    v
               Deployment
                    |
             +------+------+
             |      |      |
             v      v      v
           Node 1 Node 2 Node 3
             |      |      |
            Pod    Pod    Pod
             \      |      /
              Service A 2.0
```

---

## Docker vs Kubernetes

### Docker Alone

You think about individual machines:

```text
Developer
    |
   SSH
    |
  Server
    |
  Docker
    |
 Container
```

### Kubernetes

You think about the entire cluster:

```text
Developer
    |
 kubectl
    |
Kubernetes API
    |
 Cluster
    |
Kubernetes decides which node
    |
   Pod
```

---

## Key Difference

**Docker** lets you manage containers on a machine.

**Kubernetes** lets you manage containerized workloads across a cluster of machines.

---

## Important Commands

### Docker

```bash
docker build
docker images
docker run
docker ps
docker stop
docker start
docker pull
docker push
docker logs
docker exec
```

### Kubernetes

```bash
kubectl get nodes
kubectl get pods
kubectl get deployments
kubectl apply -f deployment.yaml
kubectl delete -f deployment.yaml
kubectl scale deployment service-a --replicas=3
kubectl logs <pod-name>
```

---

## Final Mental Model

```text
DOCKER

Developer
   |
   +-- SSH --> Server
                |
              Docker
                |
             Container
```

```text
KUBERNETES

Developer
   |
   +-- kubectl --> Kubernetes API
                         |
                       Cluster
                         |
             +-----------+-----------+
             |           |           |
           Node 1      Node 2      Node 3
             |           |           |
            Pods        Pods        Pods
```

Remember:

```text
Docker     = manage containers
Kubernetes = manage workloads across machines
kubectl    = command-line tool used to control Kubernetes
kubelet    = component running on each Kubernetes node
SSH        = remote terminal access to a server
Registry   = storage/distribution for container images
```
