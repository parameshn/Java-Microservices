# Docker Images and Kubernetes Runtime Compatibility

## Main Idea

A Kubernetes Pod does **not** need Docker to run containers.

Docker can build the container image, while Kubernetes can use another container runtime such as `containerd` or `CRI-O` to run the container.

```text
Developer
   |
   | docker build
   ↓
Container Image
   |
   | docker push
   ↓
Container Registry
   |
   ↓
Kubernetes
   |
   ↓
kubelet
   |
   ↓
Container Runtime
   |
   ├── containerd
   └── CRI-O
        |
        ↓
     Container
        |
        ↓
   Your Application
```

## Why Is This Cross-Compatible?

Docker does not create a special Docker-only application format.

It produces a **standard container image**.

The image contains things such as:

```text
restaurant:v1
├── Linux filesystem
├── Java runtime
├── Application JAR
└── Configuration
```

A compatible container runtime understands the standard image format and can turn it into a running container.

## Kubernetes and the Runtime

Kubernetes normally communicates with the container runtime through the **CRI (Container Runtime Interface)**.

```text
Kubernetes
     |
   kubelet
     |
    CRI
     |
 ┌─────────────┐
 ↓             ↓
containerd    CRI-O
 ↓             ↓
Container     Container
```

Kubernetes does not need to know the internal implementation of the runtime.

It can essentially request:

```text
"Run this container from this image."
```

The runtime performs the actual container creation and execution.

## Important Distinction

```text
Docker
  ↓
Builds container images

Registry
  ↓
Stores container images

Kubernetes
  ↓
Manages Pods

Container Runtime
  ↓
Actually creates and runs containers
```

## Example

You can build an image using Docker:

```bash
docker build -t restaurant:v1 .
```

Push it:

```bash
docker push registry.company.com/restaurant:v1
```

Then a Kubernetes node can run that image using `containerd`:

```text
Registry
   ↓
containerd
   ↓
Pod
   ↓
Container
   ↓
Spring Boot application
```

Docker does not have to be installed on that Kubernetes worker node.

## Simple Analogy

Think about a PDF:

```text
Microsoft Word
      |
      ↓
     PDF
      |
 ┌────┴────┐
 ↓         ↓
Chrome   Adobe Reader
```

The program that creates the file does not have to be the program that opens it.

Similarly:

```text
Docker
   ↓
Container Image
   ↓
containerd / CRI-O
   ↓
Running Container
```

## Easy Memory

```text
Docker      = builds the image
Registry    = stores the image
Kubernetes  = manages the Pod
Runtime     = runs the container
```

### One-line mental model

**Docker can build the image; Kubernetes can run that image using a container runtime such as containerd or CRI-O.**
