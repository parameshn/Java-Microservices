# Zero-Downtime Deployment

## Basic Idea

Kubernetes can update a running service **without stopping all Pods at once**.

```text
Old Version
v1.0.0
   ↓
Rolling Update
   ↓
New Version
v1.1.0
```

## 3-Step Deployment

### 1. Build and Push New Image

Build the new application image and push it to the registry with a new version tag.

```bash
docker build -t ftgo-restaurant-service:1.1.0.RELEASE .
docker push registry.example.com/ftgo-restaurant-service:1.1.0.RELEASE
```

Old:

```text
ftgo-restaurant-service:1.0.0.RELEASE
```

New:

```text
ftgo-restaurant-service:1.1.0.RELEASE
```

### 2. Update Deployment YAML

Change the image:

```yaml
containers:
  - name: ftgo-restaurant-service
    image: registry.example.com/ftgo-restaurant-service:1.1.0.RELEASE
```

### 3. Apply the Deployment

```bash
kubectl apply -f ftgo-restaurant-service-deployment.yml
```

Kubernetes starts a **rolling update**.

## Rolling Update

Suppose there are 3 Pods running version 1.0.0:

```text
Before:

v1.0   v1.0   v1.0
 P1     P2     P3
```

Kubernetes gradually creates new Pods:

```text
Step 1:

v1.0   v1.0   v1.1
 P1     P2     P4
```

Once the new Pod is ready:

```text
Step 2:

v1.0   v1.1   v1.1
 P1     P4     P5
```

Finally:

```text
Step 3:

v1.1   v1.1   v1.1
 P4     P5     P6
```

The old Pods are removed **gradually**, not all at once.

## Readiness Probe

Kubernetes uses the **readiness probe** to determine whether a new Pod can receive traffic.

```text
New Pod
   ↓
Starting
   ↓
Readiness Probe
   ↓
Ready? ── No ──→ Don't send traffic
   │
  Yes
   ↓
Send traffic
   ↓
Remove old Pod
```

This helps maintain availability during the rollout.

## What If the New Version Fails?

Suppose version `1.1.0` has a problem.

```text
v1.0   v1.0   v1.1 ❌
 P1     P2     P3
```

The rollout can become **stuck** because the new Pod never becomes ready.

You can:

### Fix the Deployment

Fix the YAML and apply it again:

```bash
kubectl apply -f deployment.yml
```

### Or Roll Back

```bash
kubectl rollout undo deployment ftgo-restaurant-service
```

Kubernetes uses the Deployment's rollout history to restore the previous version.

```text
v1.1.0 ❌
   ↓
rollback
   ↓
v1.0.0 ✅
```

## Important Limitation

A readiness probe only tells Kubernetes:

> **"Is this Pod ready to receive traffic?"**

It does **not** guarantee that the new application version is bug-free.

For example:

```text
New Pod
   ↓
Starts successfully
   ↓
Readiness probe passes
   ↓
Receives production traffic
   ↓
Bug appears later ❌
```

Users may already be affected.

This is why larger systems may separate:

```text
Deployment
    ↓
Get new version running
```

from:

```text
Release
    ↓
Actually expose new version to users
```

Techniques such as **service mesh traffic management, canary releases, and blue-green deployments** can be used for more controlled releases.

## Easy Memory

```text
Build new image
      ↓
Update Deployment
      ↓
kubectl apply
      ↓
Rolling Update
      ↓
New Pod becomes Ready
      ↓
Old Pod removed
      ↓
Repeat
```

> **Rolling update = gradually replace old Pods with new Pods while keeping the service available.**
