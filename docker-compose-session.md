# Docker Compose Session

## Table of Contents
1. [Docker Fundamentals Recap](#docker-fundamentals-recap)
2. [Introduction to Docker Compose](#introduction-to-docker-compose)
3. [Why Docker Compose?](#why-docker-compose)
4. [How Docker Compose Works](#how-docker-compose-works)
5. [Docker Compose File Structure](#docker-compose-file-structure)
6. [Essential Commands](#essential-commands)
7. [Common Use Cases](#common-use-cases)
8. [Best Practices](#best-practices)

---

## Docker Fundamentals Recap

Before diving into Docker Compose, let's quickly recap the core Docker concepts that form the foundation for understanding multi-container applications.

### What is Docker?

Docker is a platform that enables you to package applications and their dependencies into lightweight, portable units called **containers**. Containers run consistently across different environments, solving the "it works on my machine" problem by bundling everything needed to run an application.

### Docker Images

**Images** are read-only templates used to create containers. Think of them as blueprints or snapshots that define:
- The operating system and base software
- Application code and dependencies
- Configuration files
- Runtime settings

Images are built from `Dockerfile` instructions and stored in registries like Docker Hub. You pull images (`docker pull`) or build them (`docker build`) before running containers.

### Docker Containers

**Containers** are running instances of images. When you execute `docker run`, Docker creates a container from an image. Each container:
- Has its own isolated filesystem
- Runs in its own process space
- Can communicate with other containers through networks
- Can persist data using volumes

Containers are ephemeral by default—when stopped and removed, their changes are lost unless data is stored in volumes.

### Docker Volumes

**Volumes** provide persistent storage for containers. Unlike the container's filesystem (which is temporary), volumes:
- Persist data even after containers are removed
- Can be shared between multiple containers
- Can be backed up and restored
- Are managed by Docker and stored outside container lifecycles

You create volumes with `docker volume create` and mount them into containers to preserve important data like databases, logs, or application files.

### Docker Networks

**Networks** enable containers to communicate with each other and the outside world. Docker provides several network types:
- **Bridge networks**: Default isolated networks for containers on the same host
- **Host networks**: Containers share the host's network stack
- **Overlay networks**: Connect containers across multiple Docker hosts

Containers on the same network can discover each other by name, making it easy to connect services like web applications to databases.

---

### The Challenge: Managing Multiple Containers

When building real-world applications, you often need multiple containers working together:
- A web server container
- A database container
- A cache container (Redis)
- A background worker container

Managing these individually with `docker run` commands becomes complex—you need to create networks, volumes, set environment variables, handle dependencies, and coordinate startup order. This is where **Docker Compose** comes in.

---

## Introduction to Docker Compose

### What is Docker Compose?

- **Definition**: Docker Compose is a tool for defining and running multi-container Docker applications
- **Purpose**: Simplifies the orchestration of multiple containers that work together
- **File-based Configuration**: Uses YAML files to configure application services
- **Single Command Operations**: Start, stop, and manage entire application stacks with simple commands

### Key Concepts

- **Services**: Individual containers defined in the compose file
- **Projects**: A collection of services that work together
- **Networks**: Automatic network creation for service communication
- **Volumes**: Persistent data storage across containers

---

## Why Docker Compose?

### Problems it Solves

1. **Complex Command Lines**: Eliminates long `docker run` commands with multiple flags
2. **Service Dependencies**: Manages startup order and dependencies between containers
3. **Network Management**: Automatically creates and connects containers to networks
4. **Volume Management**: Simplifies volume creation and mounting
5. **Environment Variables**: Centralized configuration management
6. **Reproducibility**: Ensures consistent environments across different machines

### Before vs After

**Without Docker Compose:**
```bash
docker network create myapp-network
docker volume create db-data
docker run -d --name db --network myapp-network -v db-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret mysql:8
docker run -d --name app --network myapp-network -p 3000:3000 -e DB_HOST=db myapp:latest
```

**With Docker Compose:**
```bash
docker compose up -d
```

---

## How Docker Compose Works

### Architecture

1. **Compose File Parsing**: Reads `docker-compose.yml` or `docker-compose.yaml`
2. **Service Definition**: Creates service definitions from YAML configuration
3. **Network Creation**: Automatically creates a default network for services
4. **Container Orchestration**: Manages container lifecycle (start, stop, restart)
5. **Dependency Resolution**: Handles service dependencies and startup order
6. **Resource Management**: Manages volumes, networks, and container resources

### Workflow

```
docker-compose.yml → docker compose up → Container Orchestration → Running Services
```

### Default Behavior

- Creates a **default network** for all services in the compose file
- Services can reference each other by **service name** (DNS resolution)
- Containers are **prefixed** with project name (directory name by default)
- **Isolated environment** per project directory

---

## Docker Compose File Structure

### Basic Structure

```yaml
version: '3.8'  # Optional in Compose V2

services:
  service-name:
    image: image-name:tag
    ports:
      - "host-port:container-port"
    environment:
      - KEY=value
    volumes:
      - volume-name:/path/in/container

volumes:
  volume-name:

networks:
  network-name:
```

### Key Sections

#### 1. Services Section

```yaml
services:
  web:
    image: nginx:latest
    container_name: my-nginx
    ports:
      - "8080:80"
    environment:
      - NGINX_HOST=example.com
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - frontend
    depends_on:
      - api
    restart: unless-stopped
```

#### 2. Build Configuration

```yaml
services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        - BUILD_ENV=production
    image: myapp:latest
```

#### 3. Environment Variables

```yaml
services:
  db:
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=myapp
    env_file:
      - .env
      - .env.production
```

#### 4. Volumes

Docker Compose simplifies volume management by automatically creating and managing volumes defined in your compose file. Volumes persist data beyond container lifecycles and can be shared between services.

**Volume Types:**

1. **Named Volumes** (Recommended for production data)
   ```yaml
   services:
     db:
       volumes:
         - db-data:/var/lib/mysql
   
   volumes:
     db-data:
       driver: local
   ```
   - Created automatically by Docker Compose
   - Managed by Docker (stored in `/var/lib/docker/volumes/`)
   - Survive `docker compose down` (unless using `-v` flag)
   - Can be shared across multiple services

2. **Bind Mounts** (For development and configuration files)
   ```yaml
   services:
     app:
       volumes:
         - ./src:/app/src           # Mount local directory
         - ./config:/app/config      # Mount config files
         - /app/node_modules         # Anonymous volume (exclude from bind)
   ```
   - Direct mapping to host filesystem
   - Useful for live code reloading in development
   - Changes reflect immediately on host

3. **Anonymous Volumes** (Temporary data)
   ```yaml
   services:
     app:
       volumes:
         - /app/cache
   ```
   - Created automatically but not named
   - Removed when container is removed

**How Docker Compose Manages Volumes:**
- Automatically creates named volumes on `docker compose up`
- Prevents data loss when containers are recreated
- Enables data sharing between services by mounting the same volume
- Volumes persist even when services are stopped
- Use `docker compose down -v` to remove volumes

**Volume Sharing Example:**
```yaml
services:
  db:
    volumes:
      - db-data:/var/lib/mysql
  
  backup:
    volumes:
      - db-data:/backup-source  # Same volume, different mount point
  
volumes:
  db-data:
```

#### 5. Networks

Docker Compose automatically creates networks for your services, enabling seamless communication between containers. Services on the same network can discover each other by service name.

**Default Network Behavior:**
- Docker Compose creates a default bridge network for all services
- Services can communicate using their service names as hostnames
- Each project gets an isolated network (prefixed with project name)

**Custom Networks:**
```yaml
services:
  web:
    networks:
      - frontend
  api:
    networks:
      - frontend
      - backend
  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

**Network Types:**

1. **Bridge Networks** (Default)
   - Isolated network for containers on same host
   - Containers can communicate by name
   - External access via port mapping

2. **Internal Networks**
   ```yaml
   networks:
     internal:
       internal: true
   ```
   - No external internet access
   - Perfect for database containers that shouldn't be exposed

3. **External Networks**
   ```yaml
   networks:
     existing:
       external: true
       name: my-existing-network
   ```
   - Connect to networks created outside Docker Compose
   - Useful for connecting to containers managed separately

**How Docker Compose Manages Networks:**
- Creates networks automatically on `docker compose up`
- Services on same network can reach each other by service name
- DNS resolution works automatically (e.g., `db` resolves to database container)
- Networks are isolated per project (directory name)
- Removed automatically with `docker compose down` (unless external)

**Service Discovery Example:**
```yaml
services:
  web:
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    # 'db' hostname automatically resolves to db service
  db:
    image: postgres:15
    # No need to expose ports for internal communication
```

#### 6. Health Checks

```yaml
services:
  web:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

#### 7. Resource Limits

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

---

## Essential Commands

### Basic Commands

#### Start Services
```bash
# Start services in foreground
docker compose up

# Start services in detached mode
docker compose up -d

# Start specific services
docker compose up web db

# Recreate containers
docker compose up --force-recreate

# Build images before starting
docker compose up --build
```

#### Stop Services
```bash
# Stop services
docker compose stop

# Stop and remove containers
docker compose down

# Stop and remove containers, volumes, and networks
docker compose down -v

# Stop and remove images
docker compose down --rmi all
```

#### View Status
```bash
# List running services
docker compose ps

# View logs
docker compose logs

# Follow logs
docker compose logs -f

# View logs for specific service
docker compose logs web

# View last N lines
docker compose logs --tail=100
```

### Management Commands

#### Execute Commands
```bash
# Execute command in running container
docker compose exec web ls -la

# Execute command in new container
docker compose run web npm install

# Execute interactive shell
docker compose exec web sh
```

#### Build and Pull
```bash
# Build images
docker compose build

# Build without cache
docker compose build --no-cache

# Pull images
docker compose pull

# Pull and start
docker compose up --pull always
```

#### Restart and Scale
```bash
# Restart services
docker compose restart

# Restart specific service
docker compose restart web

# Scale services
docker compose up --scale web=3

# Scale multiple services
docker compose up --scale web=3 --scale worker=5
```

### Information Commands

```bash
# Validate compose file
docker compose config

# View configuration
docker compose config --services

# View service ports
docker compose port web 80

# View service logs
docker compose top
```

### Cleanup Commands

```bash
# Remove stopped containers
docker compose rm

# Remove with volumes
docker compose rm -v

# Remove unused resources
docker compose down --remove-orphans

# Prune system
docker system prune
```

---

## Common Use Cases

### 1. Web Application Stack

```yaml
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - api

  api:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

### 2. Development Environment

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    command: npm run dev
```

### 3. Microservices Architecture

```yaml
services:
  gateway:
    image: nginx
    ports:
      - "80:80"
    depends_on:
      - service1
      - service2

  service1:
    build: ./service1
    networks:
      - backend

  service2:
    build: ./service2
    networks:
      - backend

networks:
  backend:
    driver: bridge
```

### 4. Database with Admin Tool

```yaml
services:
  mysql:
    image: mysql:8
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=mydb
    volumes:
      - mysql-data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - "8080:80"
    environment:
      - PMA_HOST=mysql
    depends_on:
      - mysql

volumes:
  mysql-data:
```

### 5. CI/CD Pipeline

```yaml
services:
  test:
    build: .
    command: npm test
    volumes:
      - .:/app

  build:
    build: .
    command: npm run build
    volumes:
      - ./dist:/app/dist
```

---

## Best Practices

### 1. File Organization

- Use descriptive service names
- Group related services
- Use comments for clarity
- Keep compose files version-controlled

### 2. Environment Variables

```yaml
# Use .env file
services:
  db:
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
```

```bash
# .env file
DB_PASSWORD=secret123
```

### 3. Volume Management

```yaml
# Named volumes for data persistence
volumes:
  db-data:
    driver: local

# Bind mounts for development
volumes:
  - ./src:/app/src
```

### 4. Network Isolation

```yaml
# Separate networks for different tiers
networks:
  frontend:
  backend:
    internal: true  # No external access
```

### 5. Health Checks

```yaml
services:
  web:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### 6. Resource Limits

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G
```

### 7. Security

- Don't commit secrets in compose files
- Use secrets management
- Run containers as non-root users
- Use specific image tags (not `latest`)

### 8. Multi-Stage Compose Files

```yaml
# docker-compose.dev.yml
services:
  app:
    volumes:
      - .:/app

# docker-compose.prod.yml
services:
  app:
    build:
      context: .
      target: production
```

---

## Summary

### Key Takeaways

1. **Docker Compose** simplifies multi-container application management
2. **YAML-based** configuration for declarative service definitions
3. **Automatic networking** and DNS resolution between services
4. **Single command** operations for entire application stacks
5. **Environment-specific** configurations with multiple compose files
6. **Dependency management** with `depends_on` and health checks
7. **Volume and network** management built-in

### Next Steps

- Practice with real-world examples
- Explore Docker Compose in production (Docker Swarm)
- Learn about Docker Compose V2 improvements
- Study advanced orchestration with Kubernetes

---

## Additional Resources

- [Docker Compose Official Documentation](https://docs.docker.com/compose/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Docker Compose CLI Reference](https://docs.docker.com/compose/reference/)
- [Best Practices Guide](https://docs.docker.com/compose/production/)

---

## Practice Exercises

1. Create a WordPress site with MySQL database
2. Set up a Node.js app with Redis cache
3. Build a microservices architecture with 3 services
4. Configure a development environment with hot-reload
5. Implement health checks and dependency management

