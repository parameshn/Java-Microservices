# Kubernetes Service

A **Service** provides a stable network endpoint for one or more Pods.

Pods can be created, deleted, or replaced, so their IP addresses can change.

```text
Client
   |
   ↓
Service
   |
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

## Why Service?

Without a Service:

```text
Client → Pod IP
```

If the Pod is replaced, its IP can change.

With a Service:

```text
Client
   |
   ↓
restaurant-service:8080
   |
   ↓
Service
   |
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

The Service provides a **stable IP address and DNS name**.

## Service YAML

```yaml
apiVersion: v1
kind: Service

metadata:
  name: ftgo-restaurant-service

spec:
  ports:
    - port: 8080
      targetPort: 8080

  selector:
    app: ftgo-restaurant-service
```

### `name`

```yaml
name: ftgo-restaurant-service
```

The Service name is also used as its DNS name inside the cluster.

```text
http://ftgo-restaurant-service:8080
```

### `port`

```yaml
port: 8080
```

The port exposed by the Service.

### `targetPort`

```yaml
targetPort: 8080
```

The port on the selected Pod/container that receives the traffic.

```text
Service :8080
      ↓
Pod :8080
```

### `selector`

```yaml
selector:
  app: ftgo-restaurant-service
```

The selector tells the Service which Pods to route traffic to.

The Pods must have the matching label:

```yaml
labels:
  app: ftgo-restaurant-service
```

```text
Service
   |
   | selector:
   | app=ftgo-restaurant-service
   ↓
┌───────────────┐
│ Pod 1         │
│ Pod 2         │
│ Pod 3         │
└───────────────┘
```

## How the Service Finds Pods

Kubernetes maintains endpoint information for the Service using **EndpointSlices**.

```text
Service
   |
   ↓
EndpointSlice
   |
   ├── Pod 1 → 10.0.0.10:8080
   ├── Pod 2 → 10.0.0.11:8080
   └── Pod 3 → 10.0.0.12:8080
```

If a Pod disappears, Kubernetes updates the EndpointSlice.

```text
Pod 1 ❌

EndpointSlice
   |
   ├── Pod 2
   └── Pod 3
```

A replacement Pod can then be added.

## Service Types

### ClusterIP

Default type. Accessible from inside the cluster.

```text
Client inside cluster
        ↓
   ClusterIP Service
        ↓
       Pods
```

```yaml
type: ClusterIP
```

### NodePort

Exposes the Service through a port on each node.

```text
Node IP :30000
       ↓
NodePort Service
       ↓
      Pods
```

```yaml
type: NodePort

ports:
  - nodePort: 30000
    port: 80
    targetPort: 8080
```

### LoadBalancer

Creates/integrates with a cloud load balancer.

```text
Internet
   ↓
Cloud Load Balancer
   ↓
Kubernetes Service
   ↓
Pods
```

```yaml
type: LoadBalancer
```

## Easy Memory

> **Service = stable address in front of Pods**
>
> **Selector = identifies which Pods**
>
> **EndpointSlice = current Pod endpoints**
