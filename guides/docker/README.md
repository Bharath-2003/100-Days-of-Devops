# Docker Fundamentals for DevOps

## 🎯 Overview

Docker revolutionized application deployment by containerizing applications with all their dependencies. It's essential for modern DevOps practices.

**Duration:** 3 weeks (Weeks 13-15)
**Time Investment:** 3-4 hours/day
**Difficulty:** Intermediate
**Prerequisites:** Linux basics, Shell scripting

---

## 📚 What You'll Learn

### Week 1: Docker Basics
- Docker architecture and concepts
- Images and containers
- Docker CLI commands
- Container lifecycle management
- Basic networking

### Week 2: Docker Images
- Dockerfile best practices
- Multi-stage builds
- Image optimization
- Docker registry
- Image security

### Week 3: Advanced Docker
- Docker Compose
- Volumes and data persistence
- Networking deep dive
- Docker security
- Production best practices

---

## 🎓 Important Concepts

### 1. Docker Architecture

```
┌─────────────────────────────────────┐
│         Docker Client (CLI)         │
└──────────────┬──────────────────────┘
               │ Docker API
┌──────────────▼──────────────────────┐
│         Docker Daemon (dockerd)     │
├─────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐        │
│  │Container │  │Container │  ...   │
│  └──────────┘  └──────────┘        │
├─────────────────────────────────────┤
│         Container Runtime           │
│         (containerd, runc)          │
└─────────────────────────────────────┘
               │
┌──────────────▼──────────────────────┐
│         Host Operating System       │
└─────────────────────────────────────┘
```

**Key Components:**
- **Docker Client:** CLI tool (docker command)
- **Docker Daemon:** Background service managing containers
- **Docker Registry:** Stores Docker images (Docker Hub, ECR, etc.)
- **Docker Images:** Read-only templates
- **Docker Containers:** Running instances of images

---

### 2. Images vs Containers

```
Image (Template)              Container (Running Instance)
├─ Read-only                 ├─ Writable layer on top
├─ Layered file system       ├─ Isolated process
├─ Can be shared             ├─ Has its own network
├─ Stored in registry        ├─ Has its own filesystem
└─ Blueprint for containers  └─ Created from image
```

**Image Layers:**
```
┌─────────────────────┐
│  Application Layer  │  ← Your app
├─────────────────────┤
│  Dependencies Layer │  ← npm install, pip install
├─────────────────────┤
│  Runtime Layer      │  ← Node.js, Python
├─────────────────────┤
│  Base OS Layer      │  ← Ubuntu, Alpine
└─────────────────────┘
```

---

### 3. Dockerfile Instructions

```dockerfile
# Base image
FROM ubuntu:20.04

# Metadata
LABEL maintainer="devops@example.com"
LABEL version="1.0"

# Environment variables
ENV APP_HOME=/app
ENV NODE_ENV=production

# Working directory
WORKDIR /app

# Copy files
COPY package*.json ./
COPY . .

# Run commands
RUN apt-get update && \
    apt-get install -y nodejs npm && \
    npm install --production && \
    apt-get clean

# Expose ports
EXPOSE 3000

# Volume mount points
VOLUME ["/app/data"]

# User to run as
USER node

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

# Default command
CMD ["node", "server.js"]
```

**Key Instructions:**
- `FROM`: Base image
- `RUN`: Execute commands during build
- `COPY/ADD`: Copy files into image
- `WORKDIR`: Set working directory
- `ENV`: Set environment variables
- `EXPOSE`: Document ports
- `CMD`: Default command
- `ENTRYPOINT`: Main executable
- `VOLUME`: Mount points
- `USER`: Run as specific user

---

### 4. Container Lifecycle

```
┌─────────┐
│ Created │ ← docker create
└────┬────┘
     │ docker start
┌────▼────┐
│ Running │ ← docker run
└────┬────┘
     │ docker stop
┌────▼────┐
│ Stopped │
└────┬────┘
     │ docker rm
┌────▼────┐
│ Removed │
└─────────┘
```

**States:**
- **Created:** Container created but not started
- **Running:** Container is executing
- **Paused:** Container processes are paused
- **Stopped:** Container has exited
- **Removed:** Container deleted

---

### 5. Docker Networking

```
Bridge Network (default)
┌────────────────────────────────┐
│  Docker Host                   │
│  ┌──────────┐  ┌──────────┐  │
│  │Container1│  │Container2│  │
│  │  eth0    │  │  eth0    │  │
│  └────┬─────┘  └────┬─────┘  │
│       │             │         │
│  ┌────▼─────────────▼─────┐  │
│  │   docker0 bridge       │  │
│  └────────────┬────────────┘  │
└───────────────┼────────────────┘
                │
        ┌───────▼────────┐
        │  Host Network  │
        └────────────────┘
```

**Network Types:**
- **Bridge:** Default, isolated network
- **Host:** Use host's network directly
- **None:** No networking
- **Overlay:** Multi-host networking
- **Macvlan:** Assign MAC address to container

---

### 6. Docker Volumes

```
Volume Types:

1. Named Volume (Managed by Docker)
   docker volume create mydata
   /var/lib/docker/volumes/mydata

2. Bind Mount (Host directory)
   -v /host/path:/container/path

3. tmpfs Mount (Memory)
   --tmpfs /container/path
```

**Volume vs Bind Mount:**
```
Volume:
✓ Managed by Docker
✓ Portable across hosts
✓ Can be shared between containers
✓ Backed up easily

Bind Mount:
✓ Direct host access
✓ Good for development
✗ Host-dependent
✗ Less portable
```

---

## 💻 Hands-On Exercises

### Exercise 1: Docker Basics (45 minutes)

**Objective:** Master basic Docker commands

```bash
# 1. Check Docker installation
docker --version
docker info

# 2. Run your first container
docker run hello-world

# 3. Run interactive container
docker run -it ubuntu:20.04 bash
# Inside container:
apt-get update
apt-get install -y curl
curl https://example.com
exit

# 4. Run container in background
docker run -d --name mynginx nginx:latest

# 5. List running containers
docker ps

# 6. List all containers
docker ps -a

# 7. View container logs
docker logs mynginx
docker logs -f mynginx  # Follow logs

# 8. Execute command in running container
docker exec mynginx ls /usr/share/nginx/html
docker exec -it mynginx bash

# 9. Stop container
docker stop mynginx

# 10. Start container
docker start mynginx

# 11. Restart container
docker restart mynginx

# 12. Remove container
docker stop mynginx
docker rm mynginx

# 13. Remove container forcefully
docker rm -f mynginx

# 14. List images
docker images

# 15. Pull image
docker pull nginx:alpine

# 16. Remove image
docker rmi nginx:alpine

# 17. View container stats
docker stats

# 18. Inspect container
docker inspect mynginx

# 19. View container processes
docker top mynginx

# 20. Copy files to/from container
docker cp file.txt mynginx:/tmp/
docker cp mynginx:/tmp/file.txt ./
```

**Challenge:**
Run a MySQL container with:
- Custom root password
- Named volume for data persistence
- Port mapping to host
- Custom network

**Solution:**
```bash
# Create network
docker network create mysql-net

# Create volume
docker volume create mysql-data

# Run MySQL
docker run -d \
  --name mysql-server \
  --network mysql-net \
  -e MYSQL_ROOT_PASSWORD=secretpass \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0

# Verify
docker ps
docker logs mysql-server
docker exec -it mysql-server mysql -uroot -psecretpass -e "SHOW DATABASES;"
```

---

### Exercise 2: Building Docker Images (60 minutes)

**Objective:** Create optimized Docker images

**Task 1: Simple Node.js Application**

```bash
# 1. Create project directory
mkdir node-app && cd node-app

# 2. Create package.json
cat > package.json << 'EOF'
{
  "name": "hello-docker",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.0"
  }
}
EOF

# 3. Create server.js
cat > server.js << 'EOF'
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Docker!' });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF

# 4. Create Dockerfile (Basic)
cat > Dockerfile << 'EOF'
FROM node:16
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
EOF

# 5. Build image
docker build -t node-app:v1 .

# 6. Run container
docker run -d -p 3000:3000 --name myapp node-app:v1

# 7. Test
curl http://localhost:3000
curl http://localhost:3000/health

# 8. View image size
docker images node-app:v1
```

**Task 2: Optimized Multi-Stage Build**

```dockerfile
# Dockerfile.optimized
# Stage 1: Build
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Runtime
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
USER node
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"
CMD ["node", "server.js"]
```

```bash
# Build optimized image
docker build -f Dockerfile.optimized -t node-app:v2 .

# Compare sizes
docker images | grep node-app

# Run optimized version
docker run -d -p 3001:3000 --name myapp-v2 node-app:v2
```

**Challenge:**
Create a Dockerfile for a Python Flask application with:
- Multi-stage build
- Non-root user
- Health check
- Minimal base image (alpine)
- Production dependencies only

**Solution:**
```dockerfile
# Stage 1: Build
FROM python:3.9-alpine AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.9-alpine
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
EXPOSE 5000
RUN adduser -D appuser && chown -R appuser:appuser /app
USER appuser
HEALTHCHECK --interval=30s CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:5000/health')"
CMD ["python", "app.py"]
```

---

### Exercise 3: Docker Compose (60 minutes)

**Objective:** Orchestrate multi-container applications

**Task: WordPress with MySQL**

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: mysql:8.0
    container_name: wordpress-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - wp-network

  wordpress:
    image: wordpress:latest
    container_name: wordpress-app
    restart: always
    depends_on:
      - db
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html
    networks:
      - wp-network

volumes:
  db_data:
  wp_data:

networks:
  wp-network:
    driver: bridge
```

```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f

# List services
docker-compose ps

# Stop services
docker-compose stop

# Start services
docker-compose start

# Restart services
docker-compose restart

# Remove services
docker-compose down

# Remove services and volumes
docker-compose down -v

# Scale service
docker-compose up -d --scale wordpress=3

# Execute command in service
docker-compose exec wordpress bash

# View service logs
docker-compose logs wordpress
```

**Challenge:**
Create a docker-compose.yml for a complete application stack:
- Nginx (reverse proxy)
- Node.js API (2 instances)
- Redis (cache)
- PostgreSQL (database)
- Custom network
- Health checks

**Solution:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 3s
      retries: 3

  api:
    build: ./api
    deploy:
      replicas: 2
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/appdb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3

  postgres:
    image: postgres:14-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: appdb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

---

### Exercise 4: Docker Networking (45 minutes)

**Objective:** Understand Docker networking

```bash
# 1. List networks
docker network ls

# 2. Inspect network
docker network inspect bridge

# 3. Create custom network
docker network create my-network

# 4. Create containers in custom network
docker run -d --name web1 --network my-network nginx:alpine
docker run -d --name web2 --network my-network nginx:alpine

# 5. Test connectivity
docker exec web1 ping -c 3 web2
docker exec web2 ping -c 3 web1

# 6. Connect container to network
docker network connect my-network container_name

# 7. Disconnect container from network
docker network disconnect my-network container_name

# 8. Create network with custom subnet
docker network create --subnet=172.20.0.0/16 custom-net

# 9. Run container with specific IP
docker run -d --name web3 --network custom-net --ip 172.20.0.10 nginx:alpine

# 10. Remove network
docker network rm my-network
```

**Challenge:**
Create a three-tier application with separate networks:
- Frontend network (web servers)
- Backend network (app servers)
- Database network (databases)
- App servers connected to both frontend and backend

**Solution:**
```bash
# Create networks
docker network create frontend-net
docker network create backend-net

# Run database (backend only)
docker run -d --name postgres \
  --network backend-net \
  -e POSTGRES_PASSWORD=secret \
  postgres:14-alpine

# Run app server (both networks)
docker run -d --name api \
  --network backend-net \
  myapi:latest

docker network connect frontend-net api

# Run web server (frontend only)
docker run -d --name nginx \
  --network frontend-net \
  -p 80:80 \
  nginx:alpine

# Verify connectivity
docker exec api ping -c 2 postgres  # Should work
docker exec nginx ping -c 2 api     # Should work
docker exec nginx ping -c 2 postgres  # Should fail
```

---

### Exercise 5: Docker Volumes (45 minutes)

**Objective:** Manage data persistence

```bash
# 1. Create named volume
docker volume create mydata

# 2. List volumes
docker volume ls

# 3. Inspect volume
docker volume inspect mydata

# 4. Run container with volume
docker run -d --name db \
  -v mydata:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# 5. Write data
docker exec -it db mysql -uroot -psecret -e "CREATE DATABASE testdb;"

# 6. Remove container
docker rm -f db

# 7. Create new container with same volume
docker run -d --name db2 \
  -v mydata:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# 8. Verify data persists
docker exec -it db2 mysql -uroot -psecret -e "SHOW DATABASES;"

# 9. Bind mount (development)
docker run -d --name web \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  -p 8080:80 \
  nginx:alpine

# 10. Backup volume
docker run --rm \
  -v mydata:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/mydata-backup.tar.gz -C /source .

# 11. Restore volume
docker run --rm \
  -v mydata:/target \
  -v $(pwd):/backup \
  alpine tar xzf /backup/mydata-backup.tar.gz -C /target

# 12. Remove volume
docker volume rm mydata
```

**Challenge:**
Create a backup script that:
1. Stops container
2. Backs up volume
3. Restarts container
4. Compresses backup
5. Uploads to S3 (simulated)

**Solution:**
```bash
#!/bin/bash
CONTAINER="myapp"
VOLUME="app-data"
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d-%H%M%S)

# Stop container
echo "Stopping container..."
docker stop $CONTAINER

# Backup volume
echo "Creating backup..."
docker run --rm \
  -v $VOLUME:/source:ro \
  -v $BACKUP_DIR:/backup \
  alpine tar czf /backup/${VOLUME}-${DATE}.tar.gz -C /source .

# Restart container
echo "Restarting container..."
docker start $CONTAINER

# Cleanup old backups (keep last 7 days)
find $BACKUP_DIR -name "${VOLUME}-*.tar.gz" -mtime +7 -delete

echo "Backup complete: ${VOLUME}-${DATE}.tar.gz"
```

---

## 🎯 Practice Projects

### Project 1: Microservices Application
Build a complete microservices stack:
- API Gateway (Nginx)
- User Service (Node.js)
- Product Service (Python)
- Order Service (Go)
- Message Queue (RabbitMQ)
- Databases (PostgreSQL, MongoDB)
- Cache (Redis)
- Monitoring (Prometheus, Grafana)

### Project 2: CI/CD Pipeline
Create a Docker-based CI/CD pipeline:
- Build Docker images
- Run tests in containers
- Push to registry
- Deploy to staging
- Run integration tests
- Deploy to production

### Project 3: Development Environment
Create a complete development environment:
- Code editor (VS Code Server)
- Database (PostgreSQL)
- Cache (Redis)
- Message queue (RabbitMQ)
- Monitoring tools
- All in Docker Compose

### Project 4: Multi-Stage Build Optimization
Optimize Docker images for:
- Node.js application
- Python application
- Go application
- Java application
Compare sizes and build times

---

## 📝 Best Practices

### 1. Dockerfile Best Practices
```dockerfile
# ✓ Use specific tags
FROM node:16-alpine

# ✗ Don't use latest
FROM node:latest

# ✓ Minimize layers
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*

# ✗ Don't create unnecessary layers
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2

# ✓ Use .dockerignore
node_modules
.git
.env

# ✓ Run as non-root user
USER node

# ✓ Use multi-stage builds
FROM node:16 AS builder
# ... build steps
FROM node:16-alpine
COPY --from=builder /app/dist ./dist
```

### 2. Security Best Practices
- Use official images
- Scan images for vulnerabilities
- Don't run as root
- Use secrets management
- Keep images updated
- Minimize attack surface
- Use read-only filesystems when possible

### 3. Performance Best Practices
- Use alpine images when possible
- Leverage build cache
- Minimize image layers
- Use multi-stage builds
- Clean up in same layer
- Use .dockerignore

---

## ✅ Completion Checklist

### Week 1
- [ ] Understand Docker architecture
- [ ] Run and manage containers
- [ ] Work with images
- [ ] Understand networking basics
- [ ] Use volumes for persistence

### Week 2
- [ ] Write Dockerfiles
- [ ] Build optimized images
- [ ] Use multi-stage builds
- [ ] Push to registry
- [ ] Implement security best practices

### Week 3
- [ ] Master Docker Compose
- [ ] Understand networking deep dive
- [ ] Implement volume strategies
- [ ] Complete practice projects
- [ ] Ready for Kubernetes

---

**Next:** [Kubernetes →](../kubernetes/README.md)

**Last Updated:** June 19, 2026