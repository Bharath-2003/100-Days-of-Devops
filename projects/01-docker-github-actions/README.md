# Project 1: Deploy a Web App with Docker + GitHub Actions

## 🎯 Project Overview

**Difficulty:** Beginner  
**Duration:** 4-6 hours  
**Technologies:** Docker, GitHub Actions, CI/CD, Node.js/Python

### What You'll Build
A complete CI/CD pipeline that automatically builds, tests, and deploys a containerized web application using Docker and GitHub Actions.

### Learning Objectives
- ✅ Containerize a web application with Docker
- ✅ Write multi-stage Dockerfiles for optimization
- ✅ Set up GitHub Actions workflows
- ✅ Implement automated testing in CI/CD
- ✅ Push Docker images to Docker Hub
- ✅ Deploy containers automatically

### Prerequisites
- Basic understanding of Git and GitHub
- Docker installed locally
- GitHub account
- Docker Hub account
- Basic knowledge of web development (Node.js or Python)

---

## 📋 Project Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     GitHub Repository                        │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Source Code (app.js, Dockerfile, tests/)          │    │
│  └────────────────────────────────────────────────────┘    │
│                           │                                  │
│                           │ Push/PR                          │
│                           ▼                                  │
│  ┌────────────────────────────────────────────────────┐    │
│  │           GitHub Actions Workflow                   │    │
│  │  ┌──────────────────────────────────────────────┐  │    │
│  │  │  1. Checkout Code                            │  │    │
│  │  │  2. Run Tests                                │  │    │
│  │  │  3. Build Docker Image                       │  │    │
│  │  │  4. Push to Docker Hub                       │  │    │
│  │  │  5. Deploy (optional)                        │  │    │
│  │  └──────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                           │
                           │ Push Image
                           ▼
                  ┌─────────────────┐
                  │   Docker Hub    │
                  │  (Image Registry)│
                  └─────────────────┘
                           │
                           │ Pull Image
                           ▼
                  ┌─────────────────┐
                  │  Production     │
                  │  Server/Cloud   │
                  └─────────────────┘
```

---

## 🚀 Step-by-Step Implementation

### Phase 1: Create the Web Application

#### Step 1.1: Set Up Project Structure
```bash
# Create project directory
mkdir docker-github-actions-demo
cd docker-github-actions-demo

# Initialize Git repository
git init

# Create project structure
mkdir -p src tests .github/workflows
touch src/app.js src/package.json
touch tests/app.test.js
touch Dockerfile .dockerignore
touch .github/workflows/ci-cd.yml
touch README.md
```

#### Step 1.2: Create a Simple Node.js Application

**File: `src/package.json`**
```json
{
  "name": "docker-github-actions-demo",
  "version": "1.0.0",
  "description": "Demo app for Docker + GitHub Actions",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "jest": "^29.5.0",
    "supertest": "^6.3.3"
  }
}
```

**File: `src/app.js`**
```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json());

// Routes
app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Docker + GitHub Actions!',
    version: '1.0.0',
    timestamp: new Date().toISOString()
  });
});

app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy' });
});

app.get('/api/users', (req, res) => {
  res.json({
    users: [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' }
    ]
  });
});

// Start server
const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// Export for testing
module.exports = { app, server };
```

#### Step 1.3: Create Tests

**File: `tests/app.test.js`**
```javascript
const request = require('supertest');
const { app, server } = require('../src/app');

describe('API Endpoints', () => {
  afterAll((done) => {
    server.close(done);
  });

  test('GET / should return welcome message', async () => {
    const response = await request(app).get('/');
    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty('message');
    expect(response.body.message).toContain('Hello');
  });

  test('GET /health should return healthy status', async () => {
    const response = await request(app).get('/health');
    expect(response.status).toBe(200);
    expect(response.body.status).toBe('healthy');
  });

  test('GET /api/users should return users array', async () => {
    const response = await request(app).get('/api/users');
    expect(response.status).toBe(200);
    expect(response.body.users).toBeInstanceOf(Array);
    expect(response.body.users.length).toBeGreaterThan(0);
  });
});
```

**File: `tests/jest.config.js`**
```javascript
module.exports = {
  testEnvironment: 'node',
  coveragePathIgnorePatterns: ['/node_modules/'],
  testMatch: ['**/*.test.js']
};
```

---

### Phase 2: Containerize the Application

#### Step 2.1: Create Dockerfile (Multi-stage Build)

**File: `Dockerfile`**
```dockerfile
# Stage 1: Build stage
FROM node:18-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package files
COPY src/package*.json ./

# Install dependencies
RUN npm ci --only=production

# Stage 2: Production stage
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set working directory
WORKDIR /app

# Copy dependencies from builder
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules

# Copy application code
COPY --chown=nodejs:nodejs src/ ./

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Start application
CMD ["node", "app.js"]
```

#### Step 2.2: Create .dockerignore

**File: `.dockerignore`**
```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
tests
.github
*.md
```

#### Step 2.3: Build and Test Docker Image Locally

```bash
# Build the image
docker build -t myapp:latest .

# Run the container
docker run -d -p 3000:3000 --name myapp-container myapp:latest

# Test the application
curl http://localhost:3000
curl http://localhost:3000/health
curl http://localhost:3000/api/users

# Check logs
docker logs myapp-container

# Stop and remove container
docker stop myapp-container
docker rm myapp-container
```

---

### Phase 3: Set Up GitHub Actions CI/CD

#### Step 3.1: Create GitHub Actions Workflow

**File: `.github/workflows/ci-cd.yml`**
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  DOCKER_IMAGE: yourusername/myapp
  DOCKER_TAG: ${{ github.sha }}

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: src/package-lock.json
      
      - name: Install dependencies
        run: |
          cd src
          npm ci
      
      - name: Run tests
        run: |
          cd src
          npm test
      
      - name: Run linting (optional)
        run: |
          cd src
          npm run lint || echo "No linting configured"

  build:
    name: Build and Push Docker Image
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push'
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.DOCKER_IMAGE }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=registry,ref=${{ env.DOCKER_IMAGE }}:buildcache
          cache-to: type=registry,ref=${{ env.DOCKER_IMAGE }}:buildcache,mode=max
      
      - name: Image digest
        run: echo ${{ steps.docker_build.outputs.digest }}

  deploy:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Deploy notification
        run: |
          echo "Deploying version ${{ github.sha }}"
          echo "Image: ${{ env.DOCKER_IMAGE }}:${{ github.sha }}"
      
      # Add your deployment steps here
      # Examples:
      # - Deploy to AWS ECS
      # - Deploy to Kubernetes
      # - Deploy to Docker Swarm
      # - SSH to server and pull new image
      
      - name: Deployment success
        run: echo "Deployment completed successfully!"
```

#### Step 3.2: Create Additional Workflows

**File: `.github/workflows/docker-security-scan.yml`**
```yaml
name: Docker Security Scan

on:
  push:
    branches: [ main ]
  schedule:
    - cron: '0 0 * * 0'  # Weekly scan

jobs:
  scan:
    name: Security Scan
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Build Docker image
        run: docker build -t myapp:scan .
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:scan'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

---

### Phase 4: Configure GitHub Repository

#### Step 4.1: Set Up GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Add the following secrets:
   - `DOCKER_USERNAME`: Your Docker Hub username
   - `DOCKER_PASSWORD`: Your Docker Hub password or access token

#### Step 4.2: Push Code to GitHub

```bash
# Create .gitignore
cat > .gitignore << 'EOF'
node_modules/
npm-debug.log
.env
.DS_Store
*.log
coverage/
.vscode/
EOF

# Initialize and push
git add .
git commit -m "Initial commit: Docker + GitHub Actions setup"
git branch -M main
git remote add origin https://github.com/yourusername/docker-github-actions-demo.git
git push -u origin main
```

---

## 🧪 Testing the Pipeline

### Test 1: Verify Workflow Triggers
```bash
# Make a change
echo "# Update" >> README.md
git add README.md
git commit -m "Test CI/CD pipeline"
git push origin main
```

### Test 2: Check GitHub Actions
1. Go to your repository on GitHub
2. Click on **Actions** tab
3. Watch the workflow run
4. Verify all jobs pass (test, build, deploy)

### Test 3: Verify Docker Hub
1. Go to Docker Hub
2. Check your repository
3. Verify the new image is pushed
4. Check image tags

### Test 4: Pull and Run the Image
```bash
# Pull the image from Docker Hub
docker pull yourusername/myapp:latest

# Run the container
docker run -d -p 3000:3000 --name prod-app yourusername/myapp:latest

# Test the application
curl http://localhost:3000

# Clean up
docker stop prod-app
docker rm prod-app
```

---

## 📊 Project Enhancements

### Enhancement 1: Add Environment Variables
```yaml
# In .github/workflows/ci-cd.yml
env:
  NODE_ENV: production
  PORT: 3000
  LOG_LEVEL: info
```

### Enhancement 2: Multi-Environment Deployment
```yaml
deploy-staging:
  name: Deploy to Staging
  runs-on: ubuntu-latest
  needs: build
  if: github.ref == 'refs/heads/develop'
  environment:
    name: staging
    url: https://staging.example.com

deploy-production:
  name: Deploy to Production
  runs-on: ubuntu-latest
  needs: build
  if: github.ref == 'refs/heads/main'
  environment:
    name: production
    url: https://example.com
```

### Enhancement 3: Add Docker Compose
**File: `docker-compose.yml`**
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
```

### Enhancement 4: Add Monitoring
```javascript
// Add to app.js
app.get('/metrics', (req, res) => {
  res.json({
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    cpu: process.cpuUsage()
  });
});
```

---

## 🐛 Troubleshooting

### Issue 1: Docker Build Fails
```bash
# Check Dockerfile syntax
docker build --no-cache -t myapp:debug .

# View build logs
docker build -t myapp:debug . 2>&1 | tee build.log
```

### Issue 2: GitHub Actions Fails
```yaml
# Add debug logging
- name: Debug
  run: |
    echo "GitHub SHA: ${{ github.sha }}"
    echo "Branch: ${{ github.ref }}"
    docker images
```

### Issue 3: Docker Hub Push Fails
```bash
# Verify credentials
docker login

# Check image name
docker images | grep myapp

# Manual push test
docker push yourusername/myapp:latest
```

---

## ✅ Success Criteria

- [ ] Application runs locally
- [ ] Tests pass locally
- [ ] Docker image builds successfully
- [ ] GitHub Actions workflow runs without errors
- [ ] Image is pushed to Docker Hub
- [ ] Can pull and run image from Docker Hub
- [ ] All tests pass in CI/CD pipeline
- [ ] Security scan completes (if enabled)

---

## 📚 Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Hub](https://hub.docker.com/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

---

## 🎓 What You Learned

✅ Docker containerization best practices  
✅ Multi-stage Docker builds for optimization  
✅ GitHub Actions workflow creation  
✅ CI/CD pipeline implementation  
✅ Automated testing in pipelines  
✅ Docker image registry management  
✅ Security scanning integration  

---

## 🚀 Next Steps

1. **Project 2**: Deploy this app to Kubernetes
2. Add database integration (PostgreSQL/MongoDB)
3. Implement blue-green deployment
4. Add performance testing
5. Set up monitoring with Prometheus

---

**Project Status:** ✅ Complete  
**Last Updated:** 2024-01-16