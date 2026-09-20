# Docker: Complete Flow + Commands

## Big Picture

```text
Java Code
   ↓
Build JAR
   ↓
Dockerfile
   ↓
docker build
   ↓
Docker Image
   ↓
docker run
   ↓
Container
   ↓
Application running
```

Production:

```text
Docker Image
   ↓
docker tag
   ↓
docker push
   ↓
Container Registry
   ↓
Production / Kubernetes
   ↓
docker pull
   ↓
Container
```

## 1. Build the Java Application

```bash
./gradlew build
```

Windows:

```powershell
.\gradlew.bat build
```

Output:

```text
build/
└── libs/
    └── restaurant-service.jar
```

Flow:

```text
Java source → Gradle → JAR
```

## 2. Dockerfile

```dockerfile
FROM eclipse-temurin:17-jre

WORKDIR /app

COPY build/libs/restaurant-service.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### FROM

```dockerfile
FROM eclipse-temurin:17-jre
```

Uses Java 17 as the base image.

### WORKDIR

```dockerfile
WORKDIR /app
```

Sets `/app` as the working directory inside the image.

### COPY

```dockerfile
COPY build/libs/restaurant-service.jar app.jar
```

Copies the JAR into the image:

```text
Your computer                         Image

build/libs/
  restaurant-service.jar  ───────→   /app/app.jar
```

### EXPOSE

```dockerfile
EXPOSE 8080
```

Documents that the application uses port 8080.

It does **not** publish the port to your host.

### ENTRYPOINT

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

When the container starts:

```text
Container starts
      ↓
java -jar app.jar
      ↓
Spring Boot starts
      ↓
Application listens on 8080
```

## 3. Build the Docker Image

```bash
docker build -t restaurant-service:1.0 .
```

Breakdown:

```text
docker build       → build an image
-t                 → assign name/tag
restaurant-service → image name
1.0                → tag/version
.                  → current directory/build context
```

Check images:

```bash
docker images
```

Example:

```text
REPOSITORY           TAG       IMAGE ID
restaurant-service   1.0       abc123
```

## 4. Image vs Container

Image:

```text
restaurant-service:1.0
```

Think:

> Blueprint/package.

Container:

```bash
docker run restaurant-service:1.0
```

Creates a running instance from the image.

```text
IMAGE
restaurant-service:1.0
        │
        │ docker run
        ↓
CONTAINER
restaurant-container
```

One image can create multiple containers:

```text
             restaurant-service:1.0
                    IMAGE
               /      |      \
              ↓       ↓       ↓
         container container container
```

## 5. Run the Container

```bash
docker run -d --name restaurant-service -p 8080:8080 restaurant-service:1.0
```

### `docker run`

Creates and starts a container from an image.

### `-d`

Detached mode. Runs in the background.

### `--name`

```bash
--name restaurant-service
```

Gives the container a name.

### `-p`

```bash
-p 8080:8080
```

Means:

```text
HOST PORT : CONTAINER PORT
    8080  →      8080
```

So:

```text
Your computer
localhost:8080
      ↓
Docker
      ↓
Container:8080
      ↓
Spring Boot
```

## 6. Check Containers

Running containers:

```bash
docker ps
```

All containers, including stopped ones:

```bash
docker ps -a
```

## 7. Logs

```bash
docker logs restaurant-service
```

Follow logs:

```bash
docker logs -f restaurant-service
```

## 8. Stop and Start

Stop:

```bash
docker stop restaurant-service
```

Start an existing stopped container:

```bash
docker start restaurant-service
```

Restart:

```bash
docker restart restaurant-service
```

Important:

```text
docker stop
    ↓
Container still exists
    ↓
docker start
    ↓
Same container runs again
```

You do not need to rebuild the image.

## 9. Delete Container

```bash
docker rm restaurant-service
```

The image can still remain:

```text
restaurant-service:1.0
```

You can create another container from it.

## 10. Delete Image

```bash
docker rmi restaurant-service:1.0
```

`rmi` means remove image.

## 11. Enter a Running Container

```bash
docker exec -it restaurant-service sh
```

Meaning:

```text
docker exec → execute a command inside container
-it          → interactive terminal
sh           → start shell
```

Then:

```bash
ls
```

You may see:

```text
app.jar
```

Exit:

```bash
exit
```

## 12. Environment Variables

Example:

```bash
docker run -d   --name restaurant-service   -p 8080:8080   -e SPRING_DATASOURCE_URL=jdbc:postgresql://database:5432/restaurant   -e SPRING_DATASOURCE_USERNAME=postgres   -e SPRING_DATASOURCE_PASSWORD=password   restaurant-service:1.0
```

`-e` means environment variable.

```text
Container
│
├── SPRING_DATASOURCE_URL
├── SPRING_DATASOURCE_USERNAME
└── SPRING_DATASOURCE_PASSWORD
```

The same image can use different configuration:

```text
Same image
    ├── Development → development DB
    ├── Testing     → testing DB
    └── Production  → production DB
```

## 13. Docker Network

Create a network:

```bash
docker network create restaurant-network
```

Run PostgreSQL:

```bash
docker run -d   --name database   --network restaurant-network   -e POSTGRES_PASSWORD=password   postgres:16
```

Run the application:

```bash
docker run -d   --name restaurant-service   --network restaurant-network   -p 8080:8080   restaurant-service:1.0
```

Conceptually:

```text
Docker network
┌──────────────────────────────────┐
│                                  │
│  restaurant-service              │
│         │                        │
│         │ PostgreSQL             │
│         ↓                        │
│      database                    │
│                                  │
└──────────────────────────────────┘
```

The application can connect to:

```text
database:5432
```

not:

```text
localhost:5432
```

Inside a container, `localhost` means **that same container**.

## 14. Docker Compose

For multiple containers, use Compose.

`compose.yaml`:

```yaml
services:
  restaurant:
    image: restaurant-service:1.0
    ports:
      - "8080:8080"

  database:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: password
```

Start:

```bash
docker compose up
```

Background:

```bash
docker compose up -d
```

Stop/remove the Compose resources:

```bash
docker compose down
```

Conceptually:

```text
             Docker Compose
                   │
       ┌───────────┴───────────┐
       ↓                       ↓
Restaurant container      PostgreSQL container
       │                       │
       └──────────┬────────────┘
                  ↓
             Docker network
```

## 15. Push Image to a Registry

Suppose:

```text
registry.acme.com
```

Tag the image:

```bash
docker tag restaurant-service:1.0 registry.acme.com/restaurant-service:1.0
```

This gives the same image another name:

```text
restaurant-service:1.0
        ↓ docker tag
registry.acme.com/restaurant-service:1.0
```

`docker tag` does **not** rebuild the application.

Push:

```bash
docker push registry.acme.com/restaurant-service:1.0
```

Flow:

```text
Your computer
     │
     │ docker push
     ↓
Container Registry
     │
     └── restaurant-service:1.0
```

## 16. Pull Image

Another machine can download it:

```bash
docker pull registry.acme.com/restaurant-service:1.0
```

Then run:

```bash
docker run -d   --name restaurant-service   -p 8080:8080   registry.acme.com/restaurant-service:1.0
```

## 17. Complete Docker Lifecycle

Development:

```text
Java source
    │
    │ ./gradlew build
    ↓
JAR
    │
    ↓
Dockerfile
    │
    │ docker build
    ↓
Docker IMAGE
    │
    │ docker run
    ↓
Docker CONTAINER
    │
    ↓
Spring Boot running
```

Production:

```text
Docker IMAGE
     │
     │ docker tag
     ↓
Registry image
     │
     │ docker push
     ↓
Container Registry
     │
     │ docker pull
     ↓
Production machine
     │
     ↓
Container runtime
     │
     ↓
Container
     │
     ↓
Application
```

## 18. Most Important Docker Commands

| Command | Meaning |
|---|---|
| `docker build` | Build an image |
| `docker images` | List images |
| `docker run` | Create + start a container |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker stop` | Stop container |
| `docker start` | Start stopped container |
| `docker restart` | Restart container |
| `docker rm` | Delete container |
| `docker rmi` | Delete image |
| `docker logs` | View container logs |
| `docker exec` | Execute command inside container |
| `docker pull` | Download image from registry |
| `docker push` | Upload image to registry |
| `docker tag` | Give image another name/tag |
| `docker network create` | Create Docker network |
| `docker network ls` | List Docker networks |
| `docker compose up` | Start Compose services |
| `docker compose down` | Stop/remove Compose resources |

## 19. Five Things You Must Not Confuse

```text
Dockerfile
    ↓
Recipe / instructions

Image
    ↓
Packaged application / blueprint

Container
    ↓
Running instance of image

Registry
    ↓
Warehouse for images

Docker
    ↓
Platform/tooling that builds, stores, runs and manages them
```

Most important relationship:

```text
Dockerfile
    │
    │ docker build
    ↓
IMAGE
    │
    │ docker run
    ↓
CONTAINER
    │
    ↓
APPLICATION RUNNING
```

Production:

```text
IMAGE
  │
  │ docker push
  ↓
REGISTRY
  │
  │ docker pull
  ↓
ANOTHER MACHINE
  │
  │ docker run
  ↓
CONTAINER
```

## Quick Mental Model

Memorize:

```text
BUILD → IMAGE → RUN → CONTAINER
```

For production:

```text
BUILD → IMAGE → PUSH → REGISTRY → PULL → CONTAINER
```

This is the core Docker workflow before moving deeper into Docker Compose and Kubernetes.
