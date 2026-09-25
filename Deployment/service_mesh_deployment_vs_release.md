# 12.4.5 Using a Service Mesh to Separate Deployment from Release

## Traditional Deployment

The traditional approach is:

```text
Staging
   ↓
Test
   ↓
Deploy to Production
   ↓
Rolling Upgrade
   ↓
New Version
```

The assumption is:

> If the new service version works in staging, it will work in production.

But staging and production are often different.

```text
Staging
   ↓
Smaller environment
   ↓
Lower traffic

Production
   ↓
Larger environment
   ↓
Much higher traffic
```

Therefore, some bugs may only appear after the service is deployed to production.

---

## Separate Deployment from Release

A more reliable approach is to separate:

### Deployment

**Running the new version in the production environment.**

### Release

**Making the new version available to end users.**

The process becomes:

```text
1. Deploy new version to production
              ↓
2. Don't route user traffic to it
              ↓
3. Test the new version in production
              ↓
4. Release to a small number of users
              ↓
5. Gradually increase the number of users
              ↓
6. Eventually send all traffic to new version
```

---

## Example

Suppose the current version is:

```text
v1.0
```

Deploy the new version:

```text
v1.0
v1.1
```

Both versions are now running in production.

Initially:

```text
Users
  ↓
v1.0 ← 100%
v1.1 ← 0%
```

After testing, release v1.1 to a small percentage:

```text
Users
  ↓
┌───────────────┐
│ v1.0 → 90%    │
│ v1.1 → 10%    │
└───────────────┘
```

If everything works correctly:

```text
v1.0 → 50%
v1.1 → 50%
```

Then:

```text
v1.0 → 10%
v1.1 → 90%
```

Finally:

```text
v1.0 → 0%
v1.1 → 100%
```

If a problem occurs, traffic can be routed back to v1.0:

```text
v1.0 → 100%
v1.1 → 0%
```

---

## How Does a Service Mesh Help?

A **service mesh** is networking infrastructure that mediates communication between services and external applications.

It can provide:

- Traffic routing
- Rule-based load balancing
- Running multiple versions of a service simultaneously
- Routing different users to different service versions

Conceptually:

```text
                    Users
                      |
                      ↓
                Service Mesh
                      |
             ┌────────┴────────┐
             ↓                 ↓
          v1.0              v1.1
          90%                10%
```

The service mesh makes it easier to **separate deployment from release**.

---

## Istio

The book uses **Istio** as the service mesh example.

The important idea is:

```text
Kubernetes
    ↓
Runs multiple versions

Service Mesh
    ↓
Controls which version receives traffic
```

### Easy Memory

> **Deployment = running the service in production.**

> **Release = making the service available to end users.**

> **Service mesh = controls traffic between different versions of the service.**
