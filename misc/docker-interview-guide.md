# Docker Interview Preparation Guide

A comprehensive guide covering fundamental concepts through advanced topics for Docker interviews.

---

## Part 1: Core Concepts

### What is Docker?

Docker is a platform for developing, shipping, and running applications in containers. Containers package an application with all its dependencies, ensuring consistent behavior across different environments.

### Images vs Containers

| Aspect | Image | Container |
|--------|-------|-----------|
| Nature | Read-only template | Running instance of an image |
| State | Immutable | Mutable (can be modified) |
| Storage | Stored in registry | Exists on host system |
| Analogy | Class definition | Object instance |

**Key point**: You can create multiple containers from a single image. Changes made in a container don't affect the base image unless you explicitly commit them.

### Docker Architecture

Docker uses a client-server architecture:

- **Docker Daemon (dockerd)**: Background service that manages images, containers, networks, and volumes
- **Docker Client**: CLI tool that sends commands to the daemon via REST API
- **Docker Registry**: Storage for Docker images (Docker Hub, ECR, GCR, private registries)
- **containerd**: Container runtime that manages container lifecycle
- **runc**: Low-level runtime that creates and runs containers

### Essential Commands

```bash
# Image management
docker pull nginx:latest
docker images
docker rmi image_name
docker build -t myapp:v1 .

# Container lifecycle
docker run -d --name web nginx
docker start/stop/restart container_name
docker rm container_name
docker ps -a

# Inspection and debugging
docker logs container_name
docker exec -it container_name /bin/bash
docker inspect container_name

# Cleanup
docker system prune -a
docker volume prune
```

---

## Part 2: Dockerfile Deep Dive

### Instruction Reference

| Instruction | Purpose | Example |
|-------------|---------|---------|
| FROM | Base image | `FROM node:18-alpine` |
| WORKDIR | Set working directory | `WORKDIR /app` |
| COPY | Copy files from host | `COPY package.json .` |
| ADD | Copy with extraction/URL support | `ADD archive.tar.gz /app` |
| RUN | Execute command during build | `RUN npm install` |
| CMD | Default command (overridable) | `CMD ["node", "app.js"]` |
| ENTRYPOINT | Fixed command (args appended) | `ENTRYPOINT ["python"]` |
| ENV | Set environment variable | `ENV NODE_ENV=production` |
| ARG | Build-time variable | `ARG VERSION=1.0` |
| EXPOSE | Document port | `EXPOSE 3000` |
| VOLUME | Create mount point | `VOLUME /data` |
| USER | Set user context | `USER node` |
| HEALTHCHECK | Container health check | `HEALTHCHECK CMD curl -f http://localhost/` |

### CMD vs ENTRYPOINT

**CMD**: Provides defaults that can be completely overridden.

```dockerfile
FROM ubuntu
CMD ["echo", "Hello"]
```
```bash
docker run myimage          # Output: Hello
docker run myimage ls -la   # Output: directory listing (CMD replaced)
```

**ENTRYPOINT**: Defines the executable; CMD becomes arguments.

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello"]
```
```bash
docker run myimage          # Output: Hello
docker run myimage Goodbye  # Output: Goodbye (CMD overridden, ENTRYPOINT stays)
```

**Combined pattern** (recommended for CLI tools):

```dockerfile
ENTRYPOINT ["python", "script.py"]
CMD ["--help"]
```

### COPY vs ADD

Use **COPY** by default — it's more explicit and predictable.

Use **ADD** only when you need:
- Automatic tar extraction: `ADD archive.tar.gz /app`
- Remote URL fetching: `ADD https://example.com/file.txt /app`

### Multi-stage Builds

Reduce final image size by separating build and runtime environments:

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

**Benefits**: Build tools, source code, and dev dependencies don't end up in the final image.

---

## Part 3: Networking

### Network Types

| Type | Description | Use Case |
|------|-------------|----------|
| bridge | Default isolated network | Single-host container communication |
| host | Shares host network stack | Performance-critical apps |
| none | No networking | Security isolation |
| overlay | Multi-host networking | Docker Swarm / cluster |
| macvlan | Assigns MAC address | Legacy app integration |

### Bridge Network Deep Dive

```bash
# Create custom bridge network
docker network create --driver bridge my-network

# Run containers on network
docker run -d --name db --network my-network postgres
docker run -d --name app --network my-network myapp

# Containers can reach each other by name
# From 'app': ping db  ✓
```

**Default bridge vs custom bridge**:
- Custom bridge: Automatic DNS resolution by container name
- Default bridge: Must use `--link` (deprecated) or IP addresses

### Host Network Mode

```bash
docker run --network host nginx
```

- Container binds directly to host ports
- No port mapping needed
- Better performance (no NAT)
- Reduced isolation
- On Docker Desktop (Mac/Windows): "host" refers to the VM, not your machine

### Port Mapping

```bash
-p 8080:80          # host:container
-p 127.0.0.1:8080:80  # bind to localhost only
-p 80                 # random host port
-p 8080:80/udp        # UDP protocol
```

---

## Part 4: Storage and Volumes

### Storage Options

| Type | Managed by | Persistence | Use Case |
|------|------------|-------------|----------|
| Volumes | Docker | Yes | Production data |
| Bind mounts | Host filesystem | Yes | Development, config files |
| tmpfs | Memory | No | Sensitive temporary data |

### Volumes

```bash
# Create and use named volume
docker volume create mydata
docker run -v mydata:/app/data myimage

# Anonymous volume
docker run -v /app/data myimage

# Inspect volume
docker volume inspect mydata
```

### Bind Mounts

```bash
# Mount host directory
docker run -v /host/path:/container/path myimage

# Read-only mount
docker run -v /host/config:/app/config:ro myimage

# Using --mount (more explicit)
docker run --mount type=bind,source=/host/path,target=/container/path myimage
```

### Volume Drivers

For cloud storage, distributed systems, or special storage backends:

```bash
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.1,rw \
  --opt device=:/path/to/dir \
  nfs-volume
```

---

## Part 5: Docker Compose

### Compose File Structure

```yaml
version: '3.8'

services:
  web:
    build: ./web
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - db
    networks:
      - frontend
      - backend
    volumes:
      - ./web:/app
      - node_modules:/app/node_modules

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  db-data:
  node_modules:
```

### Key Commands

```bash
docker compose up -d          # Start services
docker compose down           # Stop and remove
docker compose down -v        # Also remove volumes
docker compose logs -f web    # Follow logs
docker compose exec web sh    # Shell into service
docker compose build          # Rebuild images
docker compose ps             # List services
```

### depends_on vs healthcheck

`depends_on` only waits for container start, not readiness:

```yaml
services:
  web:
    depends_on:
      db:
        condition: service_healthy
  
  db:
    image: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

---

## Part 6: Advanced Topics

### Layer Caching and Optimization

Each Dockerfile instruction creates a layer. Docker caches layers and reuses them when possible.

**Optimization principles**:

1. Order instructions from least to most frequently changing
2. Combine RUN commands to reduce layers
3. Use `.dockerignore` to exclude unnecessary files

**Bad example**:
```dockerfile
COPY . .
RUN npm install
```

**Good example**:
```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
```

The good example caches `npm ci` unless package.json changes.

### .dockerignore

```
node_modules
.git
.env
*.log
Dockerfile
docker-compose.yml
.DS_Store
coverage
dist
```

### Security Best Practices

**Run as non-root user**:
```dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

**Use specific image tags**:
```dockerfile
# Bad
FROM node:latest

# Good
FROM node:18.19.0-alpine3.19
```

**Scan images for vulnerabilities**:
```bash
docker scout cves myimage:tag
```

**Use read-only filesystem**:
```bash
docker run --read-only --tmpfs /tmp myimage
```

**Drop capabilities**:
```bash
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myimage
```

### Resource Limits

```bash
docker run \
  --memory=512m \
  --memory-swap=1g \
  --cpus=1.5 \
  --pids-limit=100 \
  myimage
```

In Compose:
```yaml
services:
  web:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          memory: 256M
```

### Container Logging

```bash
# View logs
docker logs --tail 100 -f container_name

# Logging drivers
docker run --log-driver=json-file --log-opt max-size=10m myimage
```

Available drivers: `json-file`, `syslog`, `journald`, `fluentd`, `awslogs`, `splunk`, `none`

### Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

```bash
# Check health status
docker inspect --format='{{.State.Health.Status}}' container_name
```

---

## Part 7: Advanced Interview Questions

### Question 1: Explain the container lifecycle and what happens when you run `docker run nginx`

**Answer**:
1. Docker client sends request to daemon
2. Daemon checks if `nginx` image exists locally
3. If not, pulls from registry (Docker Hub by default)
4. Creates a writable container layer on top of image layers
5. Allocates network interface and IP address
6. Starts the container process (nginx in foreground)
7. Container runs until main process exits

### Question 2: How does Docker achieve isolation?

**Answer**: Docker uses Linux kernel features:

- **Namespaces**: Isolate process trees (PID), network (NET), mount points (MNT), users (USER), hostname (UTS), IPC
- **cgroups**: Limit and account for resource usage (CPU, memory, I/O)
- **Union filesystem**: Layer images efficiently (overlay2)
- **Capabilities**: Fine-grained privilege control
- **seccomp**: Restrict system calls

### Question 3: What's the difference between `docker stop` and `docker kill`?

**Answer**:
- `docker stop`: Sends SIGTERM, waits grace period (default 10s), then SIGKILL
- `docker kill`: Sends SIGKILL immediately (or specified signal)

Use `stop` for graceful shutdown; `kill` for unresponsive containers.

### Question 4: How would you debug a container that keeps crashing?

**Answer**:
```bash
# Check logs
docker logs container_name

# Inspect container state
docker inspect container_name

# Check exit code
docker inspect -f '{{.State.ExitCode}}' container_name

# Run interactively to see errors
docker run -it myimage /bin/sh

# Override entrypoint to debug
docker run -it --entrypoint /bin/sh myimage

# Check resource usage
docker stats container_name
```

### Question 5: Explain Docker's copy-on-write mechanism

**Answer**:
- Image layers are read-only and shared between containers
- Container gets a thin writable layer on top
- When a file needs modification, it's copied to the writable layer first
- Only modified files consume additional space
- This makes container creation fast and storage-efficient

### Question 6: How would you reduce Docker image size?

**Answer**:
1. Use minimal base images (Alpine, distroless, scratch)
2. Multi-stage builds to exclude build dependencies
3. Combine RUN commands to reduce layers
4. Remove package manager caches: `rm -rf /var/lib/apt/lists/*`
5. Use `.dockerignore` to exclude unnecessary files
6. Don't install unnecessary packages
7. Use `--no-install-recommends` with apt

### Question 7: What happens to data when a container is removed?

**Answer**:
- Data in the writable layer is lost
- Named volumes persist (managed by Docker)
- Anonymous volumes are removed with `docker rm -v`
- Bind mounts persist (managed by host)
- tmpfs mounts are lost (memory-only)

### Question 8: Explain Docker networking between containers on different hosts

**Answer**:
- **Overlay networks**: Built-in solution for Docker Swarm, uses VXLAN
- **Service mesh**: Tools like Istio or Linkerd for Kubernetes
- **Third-party plugins**: Weave, Flannel, Calico
- **External load balancers**: Route traffic between hosts

### Question 9: How do you handle secrets in Docker?

**Answer**:

**Development**:
- Environment variables (not recommended for sensitive data)
- `.env` files (don't commit to version control)

**Production**:
- Docker Swarm secrets: `docker secret create`
- Kubernetes secrets
- External secret managers: HashiCorp Vault, AWS Secrets Manager
- Build-time secrets with BuildKit: `--secret`

```dockerfile
# BuildKit secret mount (doesn't persist in image)
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
```

### Question 10: What is a dangling image and how do you clean them?

**Answer**:
- Dangling images: Untagged images not referenced by any container
- Usually created when rebuilding with same tag
- Clean up: `docker image prune`
- Clean everything unused: `docker system prune -a`

---

## Part 8: Scenario-Based Questions

### Scenario 1: Your container works locally but fails in production

**Debugging approach**:
1. Compare Docker versions and configurations
2. Check environment variables
3. Verify network connectivity and DNS
4. Review resource limits
5. Check volume mounts and permissions
6. Compare base image versions
7. Review application logs

### Scenario 2: Design a CI/CD pipeline for Docker

**Pipeline stages**:
1. **Build**: Build Docker image with commit SHA tag
2. **Test**: Run tests in container
3. **Scan**: Security vulnerability scanning
4. **Push**: Push to registry (only on main branch)
5. **Deploy**: Update orchestrator (Kubernetes, ECS, etc.)

```yaml
# Example GitLab CI
stages:
  - build
  - test
  - scan
  - push
  - deploy

build:
  script:
    - docker build -t myapp:$CI_COMMIT_SHA .

test:
  script:
    - docker run myapp:$CI_COMMIT_SHA npm test

push:
  script:
    - docker push myapp:$CI_COMMIT_SHA
    - docker tag myapp:$CI_COMMIT_SHA myapp:latest
    - docker push myapp:latest
  only:
    - main
```

### Scenario 3: Container running out of disk space

**Solutions**:
1. Set up log rotation: `--log-opt max-size=10m --log-opt max-file=3`
2. Use volumes for large data instead of container layer
3. Clean up regularly: `docker system prune`
4. Monitor with `docker system df`
5. Use multi-stage builds for smaller images

### Scenario 4: Microservices communication pattern

```yaml
version: '3.8'

services:
  api-gateway:
    build: ./gateway
    ports:
      - "80:80"
    networks:
      - frontend

  user-service:
    build: ./user-service
    networks:
      - frontend
      - backend

  order-service:
    build: ./order-service
    networks:
      - frontend
      - backend

  database:
    image: postgres
    networks:
      - backend

networks:
  frontend:
  backend:
    internal: true  # No external access
```

---

## Quick Reference: Common Interview Mistakes

| Mistake | Correct Approach |
|---------|-----------------|
| Using `latest` tag in production | Use specific version tags |
| Running as root | Create and use non-root user |
| Storing secrets in images | Use secret management tools |
| Large images | Multi-stage builds, minimal base images |
| Not using `.dockerignore` | Always exclude unnecessary files |
| `COPY . .` before `npm install` | Copy package.json first for caching |
| Ignoring health checks | Always implement health checks |
| Not limiting resources | Set memory and CPU limits |

---

## Additional Resources

- Official Docker Documentation: https://docs.docker.com
- Docker Best Practices: https://docs.docker.com/develop/dev-best-practices/
- Dockerfile Reference: https://docs.docker.com/reference/dockerfile/
- Docker Security: https://docs.docker.com/engine/security/

---

*Good luck with your interview!*
