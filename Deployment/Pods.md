# Kubernetes Network Namespace

## What is a Network Namespace?

A **network namespace** is an isolated network environment.

It gives processes their own:

- IP address
- Network interfaces
- Ports
- Routing table

**Pod = smallest deployable unit in Kubernetes.**

## Basic Visualization

```text
Kubernetes Cluster
        ↓
Worker Node (Machine)
        ↓
       Pod
        ↓
   Container
        ↓
   Application
   

## In a Pod

All containers inside the **same Pod share one network namespace**.

```text
Pod
┌──────────────────────────────┐
│ Shared Network Namespace     │
│                              │
│ IP: 10.244.1.15              │
│                              │
│ ┌────────────┐               │
│ │ Container A│ :8080         │
│ └────────────┘               │
│                              │
│ ┌────────────┐               │
│ │ Container B│ :9090         │
│ └────────────┘               │
└──────────────────────────────┘
```

Because they share the network namespace, **Container A can reach Container B using `localhost`**:

```text
Container A
     |
     | localhost:9090
     ↓
Container B
```

## Compare Two Pods

```text
Pod 1                         Pod 2
┌─────────────────┐           ┌─────────────────┐
│ Network NS      │           │ Network NS      │
│ IP: 10.0.0.10   │           │ IP: 10.0.0.11   │
│                 │           │                 │
│ App :8080       │           │ App :8080       │
│ Sidecar :9090   │           │ Sidecar :9090   │
└─────────────────┘           └─────────────────┘
```

Pod 1's `localhost` **doesn't mean Pod 2**.

```text
Pod 1
localhost:8080 → Pod 1's container

Pod 2
localhost:8080 → Pod 2's container
```

## Why Is This Useful?

A common pattern is an application with a sidecar:

```text
Pod
┌───────────────────────────────┐
│                               │
│ App container                 │
│ :8080                         │
│       ↕ localhost             │
│ Proxy/sidecar container       │
│ :9090                         │
│                               │
└───────────────────────────────┘
```

The application and sidecar can communicate through `localhost`.

## Easy Memory

> **Containers in the same Pod share networking. Different Pods have different network identities.**
