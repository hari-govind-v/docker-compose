# Docker Compose Session

## Table of Contents
1. [Introduction to Docker Compose](#introduction-to-docker-compose)
2. [Why Docker Compose?](#why-docker-compose)
3. [How Docker Compose Works](#how-docker-compose-works)
4. [Installation and Setup](#installation-and-setup)
5. [Docker Compose File Structure](#docker-compose-file-structure)
6. [Essential Commands](#essential-commands)
7. [Advanced Features](#advanced-features)
8. [Common Use Cases](#common-use-cases)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)

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

## Installation and Setup

### Installation Methods

1. **Standalone Compose Plugin** (Recommended for Docker Desktop)
   - Included with Docker Desktop
   - Command: `docker compose` (v2 syntax)

2. **Docker Compose V2 Plugin**
   ```bash
   # Install on Linux
   sudo apt-get update
   sudo apt-get install docker-compose-plugin
   ```

3. **Docker Compose V1** (Legacy)
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/v1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   ```

### Version Check

```bash
# V2 (Plugin)
docker compose version

# V1 (Legacy)
docker-compose --version
```

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

```yaml
services:
  db:
    volumes:
      - db-data:/var/lib/mysql
      - ./backup:/backup
      - ./config:/etc/mysql/conf.d

volumes:
  db-data:
    driver: local
```

#### 5. Networks

```yaml
services:
  web:
    networks:
      - frontend
  api:
    networks:
      - frontend
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
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

## Advanced Features

### 1. Multiple Compose Files

```bash
# Use multiple compose files
docker compose -f docker-compose.yml -f docker-compose.prod.yml up

# Override with environment-specific files
docker compose -f docker-compose.yml -f docker-compose.override.yml up
```

### 2. Profiles

```yaml
services:
  web:
    profiles: ["frontend"]
  api:
    profiles: ["backend"]
  worker:
    profiles: ["worker"]
```

```bash
# Start specific profiles
docker compose --profile frontend up
```

### 3. Extends

```yaml
# docker-compose.base.yml
services:
  app:
    image: node:18
    working_dir: /app

# docker-compose.yml
services:
  web:
    extends:
      file: docker-compose.base.yml
      service: app
    ports:
      - "3000:3000"
```

### 4. Variable Substitution

```yaml
services:
  web:
    image: ${IMAGE_NAME:-nginx}:${IMAGE_TAG:-latest}
    ports:
      - "${HOST_PORT:-8080}:80"
```

### 5. Conditional Configuration

```yaml
services:
  web:
    image: nginx
    deploy:
      replicas: ${REPLICAS:-1}
```

### 6. Secrets Management

```yaml
services:
  db:
    secrets:
      - db_password
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### 7. Configs

```yaml
services:
  web:
    configs:
      - nginx_config

configs:
  nginx_config:
    file: ./nginx.conf
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

## Troubleshooting

### Common Issues

#### 1. Port Already in Use
```bash
# Check what's using the port
sudo lsof -i :8080

# Change port in compose file
ports:
  - "8081:80"  # Use different host port
```

#### 2. Service Dependencies
```yaml
# Use health checks with depends_on
depends_on:
  db:
    condition: service_healthy

services:
  db:
    healthcheck:
      test: ["CMD", "pg_isready"]
```

#### 3. Volume Permissions
```yaml
# Set user in service
services:
  app:
    user: "1000:1000"
```

#### 4. Network Issues
```bash
# Inspect network
docker network inspect docker_default

# Check service connectivity
docker compose exec web ping db
```

#### 5. Build Context Issues
```yaml
# Ensure correct build context
build:
  context: ./app  # Relative to compose file location
  dockerfile: Dockerfile
```

### Debugging Commands

```bash
# Validate configuration
docker compose config

# View service logs
docker compose logs -f service-name

# Inspect running container
docker compose exec service-name sh

# Check service status
docker compose ps

# View resource usage
docker stats $(docker compose ps -q)
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

