# Docker to Kubernetes: Complete Flow

## 1. Developer's Laptop

You start with your Spring Boot project:

```text
YOUR LAPTOP
│
└── restaurant-service/
    │
    ├── src/
    │   └── main/
    │       └── java/
    │           └── RestaurantController.java
    │
    ├── build.gradle
    ├── application.properties
    └── Dockerfile
```

You write normal Java/Spring Boot code.

```java
@RestController
public class RestaurantController {

    @GetMapping("/restaurants")
    public String getRestaurants() {
        return "Restaurant Service";
    }
}
```

At this point:

```text
Java source code
       ↓
Not a Docker image
Not a Docker container
```

---

## 2. Build the Application

Run:

```bash
./gradlew build
```

Gradle compiles and packages the application:

```text
Java source
     │
     │ Gradle
     ▼
Compile
     │
     ▼
Test
     │
     ▼
JAR
```

You may get:

```text
build/
└── libs/
    └── restaurant-service.jar
```

The JAR is your Java application packaged for execution.

It is **not yet a Docker image**.

---

## 3. Create the Dockerfile

The Dockerfile tells Docker how to package your application.

Example:

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY build/libs/restaurant-service.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Meaning:

```text
FROM
  ↓
Start with a Java runtime

COPY
  ↓
Put your JAR into the image

ENTRYPOINT
  ↓
Tell the container what to run
```

The Dockerfile is essentially a **recipe/instruction sheet** for creating the image.

---

## 4. Build the Docker Image

Run:

```bash
docker build -t restaurant-service:1.0 .
```

Docker reads the Dockerfile:

```text
Dockerfile
    │
    ▼
FROM Java image
    │
    ▼
COPY your JAR
    │
    ▼
EXPOSE 8080
    │
    ▼
ENTRYPOINT
    │
    ▼
Docker IMAGE
```

Conceptually:

```text
┌─────────────────────────────┐
│ Docker IMAGE                │
│                             │
│ Linux/base filesystem       │
│ Java 17                     │
│ restaurant-service.jar      │
│ startup instructions        │
└─────────────────────────────┘
```

The image is named:

```text
restaurant-service:1.0
```

Where:

```text
restaurant-service
       ↑
   image/repository name

1.0
 ↑
tag/version
```

---

## 5. What Is Inside the Image?

Conceptually:

```text
restaurant-service:1.0
│
├── Linux/base filesystem
├── Java 17 runtime
├── application libraries
├── restaurant-service.jar
├── configuration
└── startup instructions
```

This is why the container does not need Java manually installed on the host.

The required Java runtime is packaged into the image.

---

## 6. Run the Image Locally

You can test the image on your laptop:

```bash
docker run -p 8080:8080 restaurant-service:1.0
```

Docker takes:

```text
IMAGE
restaurant-service:1.0
        │
        │ docker run
        ▼
CONTAINER
```

Now:

```text
Your laptop
     │
     │ localhost:8080
     ▼
┌─────────────────────────┐
│ Docker Container        │
│                         │
│ Java 17                 │
│ Spring Boot             │
│ restaurant-service.jar  │
│                         │
│ :8080                   │
└─────────────────────────┘
```

The application is now actually running inside the container.

---

# 7. Push the Image to a Registry

Suppose your company has:

```text
registry.company.com
```

Tag the image:

```bash
docker tag restaurant-service:1.0 \
  registry.company.com/restaurant-service:1.0
```

The full image reference is:

```text
registry.company.com/restaurant-service:1.0
│                  │                 │
│                  │                 └── version/tag
│                  └── image/repository
└── registry
```

Then push:

```bash
docker push registry.company.com/restaurant-service:1.0
```

The image is now stored in the registry:

```text
YOUR LAPTOP
     │
     │ docker push
     ▼
┌───────────────────────────┐
│ Container Registry        │
│                           │
│ restaurant-service:1.0    │
└───────────────────────────┘
```

Think of a registry as a **warehouse for Docker images**.

---

# 8. Why Do We Need a Registry?

Production machines are usually somewhere else.

For example:

```text
Developer laptop
       │
       │
       ▼
Container Registry
       │
       ├──────────┐
       ▼          ▼
   Machine 1   Machine 2
```

The production machines don't need your entire source-code project.

They need the **built Docker image**.

```text
Source code
    ↓
Build
    ↓
Docker image
    ↓
Registry
    ↓
Production
```

---

# 9. Kubernetes Enters

Suppose the company uses Kubernetes.

Kubernetes manages workloads across multiple machines:

```text
                    KUBERNETES
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
      Machine 1      Machine 2      Machine 3
          │              │              │
      Container      Container      Container
```

Kubernetes can be told:

> "I need 3 instances of restaurant-service."

It can arrange for:

```text
Machine 1
└── Restaurant Container #1

Machine 2
└── Restaurant Container #2

Machine 3
└── Restaurant Container #3
```

---

# 10. Where Does the Image Come From?

If a production machine doesn't already have the image, the container runtime pulls it from the registry.

Conceptually:

```text
                 REGISTRY
                     │
                     │ pull
                     ▼
              ┌─────────────┐
              │  Machine 1  │
              │             │
              │ Container   │
              │ Runtime     │
              └──────┬──────┘
                     │
                     ▼
                Container
```

The same can happen on Machine 2 and Machine 3 when they need the image.

---

# 11. Image → Container

The image contains the application and its runtime:

```text
IMAGE
│
├── Java
├── Spring Boot application
├── libraries
└── startup command
```

The runtime creates a container:

```text
IMAGE
  │
  │ create/start
  ▼
CONTAINER
  │
  ▼
java -jar app.jar
  │
  ▼
Spring Boot running
```

---

# 12. Why Multiple Containers?

Suppose one restaurant-service container cannot handle all the traffic.

You can run multiple instances:

```text
                  RESTAURANT SERVICE
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
          Container   Container   Container
             #1          #2          #3
```

All three can come from the same image:

```text
restaurant-service:1.0
```

Important distinction:

```text
IMAGE
─────
One packaged application

       │
       ├──────→ Container 1
       ├──────→ Container 2
       └──────→ Container 3
```

**One image can be used to create many containers.**

---

# 13. What If a Container Crashes?

Suppose Kubernetes expects 3 instances:

```text
Desired state:
3 containers

Actual state:
2 containers
```

Kubernetes can create a replacement:

```text
Before:

Container 1 ✓
Container 2 ✗
Container 3 ✓

        ↓ Kubernetes

After:

Container 1 ✓
Container 4 ✓
Container 3 ✓
```

The replacement uses the same image:

```text
restaurant-service:1.0
```

---

# 14. What If a Machine Dies?

Suppose:

```text
Machine 1              Machine 2              Machine 3
┌─────────┐            ┌─────────┐            ┌─────────┐
│Container│            │Container│            │Container│
│   #1    │            │   #2    │            │   #3    │
└─────────┘            └────X────┘            └─────────┘
                              💥
```

Kubernetes can schedule replacement workloads on another available machine:

```text
Machine 1              Machine 2              Machine 3
┌─────────┐            ┌─────────┐            ┌─────────┐
│Container│            │   💥    │            │Container│
│   #1    │            │         │            │   #3    │
└─────────┘            └─────────┘            │   #4    │
                                              └─────────┘
```

Container #4 uses:

```text
restaurant-service:1.0
```

---

# 15. Complete Journey

```text
┌─────────────────────────────────────────────┐
│              DEVELOPER LAPTOP              │
│                                             │
│  Java/Spring Boot source code               │
│              │                              │
│              ▼                              │
│       Gradle/Maven build                    │
│              │                              │
│              ▼                              │
│          application.jar                    │
│              │                              │
│              ▼                              │
│          Dockerfile                         │
│              │                              │
│              ▼                              │
│         docker build                        │
│              │                              │
│              ▼                              │
│      Docker Image :1.0                      │
│              │                              │
└──────────────┼──────────────────────────────┘
               │
               │ docker push
               ▼
┌─────────────────────────────────────────────┐
│             CONTAINER REGISTRY              │
│                                             │
│    restaurant-service:1.0                   │
│                                             │
└──────────────────┬──────────────────────────┘
                   │
                   │ pull
                   ▼
┌─────────────────────────────────────────────┐
│                 KUBERNETES                 │
│                                             │
│       "I need 3 instances"                  │
│                    │                        │
│        ┌───────────┼───────────┐            │
│        ▼           ▼           ▼            │
│     Machine 1   Machine 2   Machine 3       │
│        │           │           │            │
│        ▼           ▼           ▼            │
│     Container   Container   Container       │
│        #1          #2          #3           │
│                                             │
└─────────────────────────────────────────────┘
```

## The One-Sentence Mental Model

> **Developer writes code → builds a Docker image → pushes the image to a registry → Kubernetes pulls that image and creates/manages containers from it on production machines.**

## Important Modern Kubernetes Detail

Kubernetes does **not necessarily use Docker** to run containers anymore.

Modern Kubernetes commonly uses **containerd** or another CRI-compatible container runtime.

The simplified architecture is:

```text
Developer
   ↓
Dockerfile
   ↓
Docker Image
   ↓
Container Registry
   ↓
Kubernetes
   ↓
Container Runtime
   ↓
Containers
```

### Key Terms

| Term | Meaning |
|---|---|
| **Dockerfile** | Instructions for building an image |
| **Docker Image** | Packaged application + runtime + dependencies |
| **Container** | Running instance of an image |
| **Registry** | Storage/warehouse for images |
| **Docker** | Tool/platform for building and managing images/containers |
| **Kubernetes** | Orchestrator that manages workloads across machines |
| **Container Runtime** | Software that actually creates/runs containers |
