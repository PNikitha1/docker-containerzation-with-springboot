# Demo Spring Boot Application

This repository contains a small Spring Boot application (Java 17) with a `User` entity, a JPA repository, a service, and a REST controller. The project is configured to use an in-memory H2 database for development.

This README explains how to build and run the application locally and via Docker (WSL and Windows instructions included).

## Prerequisites

- Java 17 (JDK) installed if you plan to run locally.
- Docker Desktop (recommended) or Docker Engine available if you plan to build/run Docker images.
- (Optional) WSL2 with your distro (Ubuntu) if you prefer building inside WSL.

Note: The repository includes a Maven wrapper (`mvnw`) so Maven isn't required system-wide.

## Build locally (recommended)

1. Open a terminal in the project root (where `pom.xml` is).

On Windows PowerShell:

```powershell
cd D:\demo
.\mvnw.cmd -DskipTests package
```

On WSL (Ubuntu) with JDK 17 installed:

```bash
cd /mnt/d/demo
./mvnw -DskipTests package
```

After a successful build the runnable jar will be under `target/`, e.g. `target/demo-0.0.1-SNAPSHOT.jar`.

## Run locally (jar)

```bash
# from project root
java -jar target/demo-0.0.1-SNAPSHOT.jar
```

This starts the application on port 8080 by default.

## Run via Maven (dev)

```bash
# runs the Spring Boot app using the embedded plugin
./mvnw spring-boot:run
```

## REST endpoints

- GET  /users        — list users
- GET  /users/{id}   — get a user
- POST /users        — create a user (JSON body)
- PUT  /users/{id}   — update a user (JSON body)
- DELETE /users/{id} — delete a user

Example POST body:

```json
{
  "name": "Alice",
  "email": "alice@example.com"
}
```

## H2 Console

While the app runs, open the H2 console at:

http://localhost:8080/h2-console

JDBC URL: `jdbc:h2:mem:demo-db`
User: `sa`
Password: (leave empty)

Note: the `User` entity is mapped to a table named `users` to avoid conflicts with H2's reserved names.

## Docker

There are two recommended Docker workflows.

### Option A — Build jar locally and build a runtime image (recommended)

1. Build jar locally (see "Build locally" above).
2. Use the runtime-only Dockerfile (example provided in this repo). Build and run:

```bash
# from project root (Windows PowerShell)
docker build -t demo-app:latest .

docker run --rm -p 8080:8080 demo-app:latest
```

This method is fast and reliable because it avoids running Maven inside the container.

### Option B — Multi-stage Docker build (build inside container)

The provided `Dockerfile` supports a multi-stage build (it runs `mvn` inside the build stage and copies the jar into a slim runtime image). If your environment allows Docker builds with network access, you can run:

```bash
docker build -t demo-app:latest .
```

If the `mvn` step fails during the Docker build (common with network/proxy issues), prefer Option A.

### WSL notes

- If you run Docker Desktop, enable WSL integration for your distro and run `docker build` from WSL — Docker Desktop will handle the daemon.
- If you install Docker Engine inside WSL directly, ensure `dockerd` is running (or systemd is enabled) before running `docker build`.

## Troubleshooting

- `The JAVA_HOME environment variable is not defined correctly`: install JDK 17 in WSL or set `JAVA_HOME` to a JDK 17 path. Example (WSL):

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk
JAVA_BIN_PATH=$(readlink -f "$(which java)")
JAVA_HOME_DIR=$(dirname "$(dirname "$JAVA_BIN_PATH")")
echo "export JAVA_HOME=$JAVA_HOME_DIR" >> ~/.profile
source ~/.profile
```

- `SQLGrammarException` when inserting into `user`: fixed by mapping entity to table `users` (annotation `@Table(name = "users")`) because `USER` is a reserved name in some databases.

- `Cannot connect to the Docker daemon`: ensure Docker Desktop is running or the Docker daemon is started inside WSL.

## Development tips

- Use the H2 console to inspect the in-memory database while the app is running.
- If you want DTOs, request validation, or paging, add them to the controller/service layers.

## Contact / next steps

If you want, I can:
- Run the build locally and report results, or
- Help you run the Docker build from WSL (walk through JDK/Docker steps), or
- Add a `Dockerfile.runtime` that expects a locally-built jar and is simpler to use.

Pick one and I’ll provide exact commands for your environment.
