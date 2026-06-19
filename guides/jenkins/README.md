# Jenkins for DevOps

## 🎯 Overview

Jenkins is the leading open-source automation server that enables developers to build, test, and deploy their software reliably. For DevOps engineers, mastering Jenkins is essential for implementing robust CI/CD pipelines.

**Duration:** 1 week (Week 12)
**Time Investment:** 4-5 hours/day
**Difficulty:** Beginner to Advanced
**Prerequisites:** Basic understanding of Git, Linux, and software development

---

## 📚 What You'll Learn

### Day 1: Jenkins Fundamentals
- Jenkins installation and setup
- Understanding Jenkins architecture
- Basic job configuration
- Build triggers
- Workspace management

### Day 2: Jenkins Pipelines
- Declarative vs Scripted pipelines
- Jenkinsfile syntax
- Pipeline stages and steps
- Environment variables
- Pipeline best practices

### Day 3: Advanced Pipeline Features
- Parallel execution
- Matrix builds
- Shared libraries
- Pipeline templates
- Error handling

### Day 4: Integration and Plugins
- Git integration
- Docker integration
- Kubernetes deployment
- Slack notifications
- SonarQube integration

### Day 5: Security and Access Control
- User authentication
- Role-based access control
- Credentials management
- Security best practices
- Audit logging

### Day 6: Distributed Builds
- Master-agent architecture
- Agent configuration
- Cloud agents (AWS, Azure, GCP)
- Docker agents
- Kubernetes agents

### Day 7: Monitoring and Optimization
- Jenkins monitoring
- Performance tuning
- Backup and restore
- High availability
- Troubleshooting

---

## 🎓 Important Concepts

### 1. Jenkins Architecture

```
Jenkins Architecture
├─ Master (Controller)
│  ├─ Web UI
│  ├─ Job Scheduler
│  ├─ Build Dispatcher
│  └─ Plugin Manager
│
├─ Agents (Workers)
│  ├─ Permanent Agents
│  ├─ Cloud Agents
│  ├─ Docker Agents
│  └─ Kubernetes Agents
│
├─ Workspace
│  ├─ Source Code
│  ├─ Build Artifacts
│  └─ Test Results
│
└─ Storage
   ├─ Job Configurations
   ├─ Build History
   └─ Plugins
```

**Key Components:**
- **Master:** Orchestrates builds, manages configuration
- **Agent:** Executes build jobs
- **Executor:** Thread that runs builds
- **Workspace:** Directory where builds happen
- **Job:** Unit of work (build, test, deploy)

---

### 2. Pipeline Types

```
Declarative Pipeline
├─ pipeline { }
├─ agent { }
├─ stages { }
│  └─ stage { }
│     └─ steps { }
├─ post { }
└─ environment { }

Scripted Pipeline
├─ node { }
├─ stage { }
├─ Groovy code
└─ More flexibility
```

**Declarative Pipeline Example:**
```groovy
pipeline {
    agent any
    
    environment {
        APP_NAME = 'myapp'
        VERSION = '1.0.0'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

**Scripted Pipeline Example:**
```groovy
node {
    try {
        stage('Build') {
            checkout scm
            sh 'mvn clean package'
        }
        
        stage('Test') {
            sh 'mvn test'
        }
        
        stage('Deploy') {
            sh './deploy.sh'
        }
        
        currentBuild.result = 'SUCCESS'
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
```

---

### 3. Pipeline Syntax

```
Pipeline Structure
├─ agent (where to run)
│  ├─ any
│  ├─ none
│  ├─ label 'linux'
│  ├─ docker { image 'node:16' }
│  └─ kubernetes { yaml '...' }
│
├─ stages (what to do)
│  └─ stage('name') {
│     ├─ when (conditions)
│     ├─ steps (actions)
│     └─ post (cleanup)
│  }
│
├─ environment (variables)
│  ├─ Global
│  └─ Stage-specific
│
├─ parameters (inputs)
│  ├─ string
│  ├─ choice
│  ├─ boolean
│  └─ password
│
├─ triggers (when to run)
│  ├─ cron
│  ├─ pollSCM
│  └─ upstream
│
└─ post (after build)
   ├─ always
   ├─ success
   ├─ failure
   ├─ unstable
   └─ changed
```

**Advanced Pipeline Features:**
```groovy
pipeline {
    agent none
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'])
        string(name: 'VERSION', defaultValue: '1.0.0')
        booleanParam(name: 'RUN_TESTS', defaultValue: true)
    }
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        APP_NAME = 'myapp'
    }
    
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'maven:3.8-jdk-11'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package'
                archiveArtifacts artifacts: 'target/*.jar'
            }
        }
        
        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn verify'
                    }
                }
            }
        }
        
        stage('Deploy') {
            agent {
                label 'deploy-agent'
            }
            when {
                branch 'main'
            }
            steps {
                script {
                    def version = params.VERSION
                    def env = params.ENVIRONMENT
                    sh "kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${version} -n ${env}"
                }
            }
        }
    }
    
    post {
        always {
            junit '**/target/surefire-reports/*.xml'
            cleanWs()
        }
        success {
            slackSend color: 'good', message: "Build ${env.BUILD_NUMBER} succeeded"
        }
        failure {
            slackSend color: 'danger', message: "Build ${env.BUILD_NUMBER} failed"
        }
    }
}
```

---

### 4. Shared Libraries

```
Shared Library Structure
├─ vars/
│  ├─ buildApp.groovy
│  ├─ deployApp.groovy
│  └─ notifySlack.groovy
│
├─ src/
│  └─ org/company/
│     └─ Utils.groovy
│
└─ resources/
   └─ templates/
      └─ Dockerfile
```

**Shared Library Example:**
```groovy
// vars/buildApp.groovy
def call(Map config) {
    pipeline {
        agent any
        stages {
            stage('Build') {
                steps {
                    sh "mvn clean package -DskipTests=${config.skipTests}"
                }
            }
            stage('Test') {
                when {
                    expression { !config.skipTests }
                }
                steps {
                    sh 'mvn test'
                }
            }
        }
    }
}

// Usage in Jenkinsfile
@Library('my-shared-library') _
buildApp(skipTests: false)
```

---

### 5. Credentials Management

```
Credential Types
├─ Username with password
├─ SSH Username with private key
├─ Secret text
├─ Secret file
├─ Certificate
└─ Docker credentials
```

**Using Credentials:**
```groovy
pipeline {
    agent any
    
    environment {
        // Credential binding
        DOCKER_CREDS = credentials('docker-hub-credentials')
        AWS_CREDS = credentials('aws-credentials')
    }
    
    stages {
        stage('Build') {
            steps {
                // Using username/password
                withCredentials([usernamePassword(
                    credentialsId: 'github-credentials',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh 'git clone https://${GIT_USER}:${GIT_PASS}@github.com/user/repo.git'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                // Using SSH key
                sshagent(['ssh-credentials']) {
                    sh 'ssh user@server "deploy.sh"'
                }
            }
        }
        
        stage('Push Image') {
            steps {
                // Using Docker credentials
                sh 'echo $DOCKER_CREDS_PSW | docker login -u $DOCKER_CREDS_USR --password-stdin'
                sh 'docker push myimage:latest'
            }
        }
    }
}
```

---

### 6. Build Triggers

```
Trigger Types
├─ Manual (Build Now)
├─ SCM Polling (pollSCM)
├─ Webhooks (GitHub, GitLab)
├─ Scheduled (cron)
├─ Upstream (after other jobs)
└─ Remote (API trigger)
```

**Trigger Configuration:**
```groovy
pipeline {
    agent any
    
    triggers {
        // Poll SCM every 5 minutes
        pollSCM('H/5 * * * *')
        
        // Run daily at 2 AM
        cron('0 2 * * *')
        
        // Trigger after upstream job
        upstream(upstreamProjects: 'build-job', threshold: hudson.model.Result.SUCCESS)
    }
    
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
    }
}
```

**Webhook Configuration:**
```bash
# GitHub webhook URL
https://jenkins.example.com/github-webhook/

# GitLab webhook URL
https://jenkins.example.com/project/my-project

# Generic webhook
https://jenkins.example.com/generic-webhook-trigger/invoke?token=SECRET_TOKEN
```

---

### 7. Agent Configuration

```
Agent Types
├─ Permanent Agents
│  ├─ SSH connection
│  ├─ JNLP connection
│  └─ Manual launch
│
├─ Cloud Agents
│  ├─ AWS EC2
│  ├─ Azure VM
│  ├─ GCP Compute
│  └─ DigitalOcean
│
├─ Container Agents
│  ├─ Docker
│  └─ Kubernetes
│
└─ Dynamic Agents
   ├─ On-demand creation
   └─ Auto-scaling
```

**Agent Configuration Examples:**
```groovy
// Docker agent
pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
}

// Kubernetes agent
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8-jdk-11
    command: ['cat']
    tty: true
  - name: docker
    image: docker:latest
    command: ['cat']
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
'''
        }
    }
}

// Label-based agent
pipeline {
    agent {
        label 'linux && docker'
    }
}
```

---

## 🔨 Hands-On Exercises

### Exercise 1: Install and Configure Jenkins (45 minutes)

**Objective:** Set up Jenkins with Docker

```bash
# 1. Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    privileged: true
    user: root
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins-data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false
      - CASC_JENKINS_CONFIG=/var/jenkins_home/casc.yaml

volumes:
  jenkins-data:
EOF

# 2. Start Jenkins
docker-compose up -d

# 3. Get initial admin password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# 4. Access Jenkins
echo "Jenkins URL: http://localhost:8080"

# 5. Install Docker in Jenkins container
docker exec -u root jenkins bash -c "
  apt-get update && \
  apt-get install -y docker.io && \
  usermod -aG docker jenkins
"

# 6. Restart Jenkins
docker-compose restart jenkins

# 7. Install Jenkins CLI
wget http://localhost:8080/jnlpJars/jenkins-cli.jar

# 8. Install plugins via CLI
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth admin:password install-plugin \
  git \
  docker-workflow \
  kubernetes \
  pipeline-stage-view \
  slack \
  sonar

# 9. Create Configuration as Code file
cat > casc.yaml << 'EOF'
jenkins:
  systemMessage: "Jenkins configured automatically by JCasC"
  numExecutors: 2
  mode: NORMAL
  
  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: "admin"
          password: "admin123"
  
  authorizationStrategy:
    globalMatrix:
      permissions:
        - "Overall/Administer:admin"
        - "Overall/Read:authenticated"

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              scope: GLOBAL
              id: "github-credentials"
              username: "your-username"
              password: "your-token"

unclassified:
  location:
    url: "http://localhost:8080/"
EOF

# 10. Reload configuration
curl -X POST http://admin:admin123@localhost:8080/reload
```

**Challenge:**
Set up Jenkins with:
1. HTTPS enabled
2. LDAP authentication
3. Multiple agents
4. Backup configuration

---

### Exercise 2: Create First Pipeline (60 minutes)

**Objective:** Build a complete CI/CD pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        APP_NAME = 'myapp'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_IMAGE = "${DOCKER_REGISTRY}/${APP_NAME}"
    }
    
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Deployment environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests')
        string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version to deploy')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                }
            }
        }
        
        stage('Build') {
            steps {
                script {
                    echo "Building ${APP_NAME}:${params.VERSION}"
                    sh '''
                        docker build \
                          -t ${DOCKER_IMAGE}:${VERSION} \
                          -t ${DOCKER_IMAGE}:${GIT_COMMIT_SHORT} \
                          -t ${DOCKER_IMAGE}:latest \
                          .
                    '''
                }
            }
        }
        
        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'docker run --rm ${DOCKER_IMAGE}:${VERSION} npm test'
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'docker run --rm ${DOCKER_IMAGE}:${VERSION} npm run lint'
                    }
                }
                stage('Security Scan') {
                    steps {
                        sh 'trivy image ${DOCKER_IMAGE}:${VERSION}'
                    }
                }
            }
        }
        
        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${VERSION}
                        docker push ${DOCKER_IMAGE}:${GIT_COMMIT_SHORT}
                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def env = params.ENVIRONMENT
                    echo "Deploying to ${env}"
                    
                    sh """
                        kubectl set image deployment/${APP_NAME} \
                          ${APP_NAME}=${DOCKER_IMAGE}:${VERSION} \
                          -n ${env}
                        
                        kubectl rollout status deployment/${APP_NAME} -n ${env}
                    """
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
            // Send notification
        }
        failure {
            echo 'Pipeline failed!'
            // Send alert
        }
    }
}
```

**Challenge:**
Enhance pipeline with:
1. Code quality checks
2. Performance testing
3. Automated rollback
4. Multi-environment deployment

---

### Exercise 3: Implement Shared Library (60 minutes)

**Objective:** Create reusable pipeline components

```bash
# 1. Create shared library repository
mkdir jenkins-shared-library
cd jenkins-shared-library

# 2. Create directory structure
mkdir -p vars src/org/company resources

# 3. Create buildDockerImage.groovy
cat > vars/buildDockerImage.groovy << 'EOF'
def call(Map config) {
    def imageName = config.imageName
    def imageTag = config.imageTag ?: 'latest'
    def dockerfile = config.dockerfile ?: 'Dockerfile'
    def context = config.context ?: '.'
    
    echo "Building Docker image: ${imageName}:${imageTag}"
    
    sh """
        docker build \
          -f ${dockerfile} \
          -t ${imageName}:${imageTag} \
          ${context}
    """
    
    return "${imageName}:${imageTag}"
}
EOF

# 4. Create deployToKubernetes.groovy
cat > vars/deployToKubernetes.groovy << 'EOF'
def call(Map config) {
    def deployment = config.deployment
    def namespace = config.namespace
    def image = config.image
    def timeout = config.timeout ?: '5m'
    
    echo "Deploying ${image} to ${namespace}"
    
    sh """
        kubectl set image deployment/${deployment} \
          ${deployment}=${image} \
          -n ${namespace}
        
        kubectl rollout status deployment/${deployment} \
          -n ${namespace} \
          --timeout=${timeout}
    """
}
EOF

# 5. Create notifySlack.groovy
cat > vars/notifySlack.groovy << 'EOF'
def call(Map config) {
    def channel = config.channel ?: '#builds'
    def message = config.message
    def color = config.color ?: 'good'
    
    slackSend(
        channel: channel,
        color: color,
        message: """
            *${env.JOB_NAME}* - Build #${env.BUILD_NUMBER}
            ${message}
            <${env.BUILD_URL}|View Build>
        """
    )
}
EOF

# 6. Create utility class
cat > src/org/company/Utils.groovy << 'EOF'
package org.company

class Utils implements Serializable {
    def steps
    
    Utils(steps) {
        this.steps = steps
    }
    
    def getGitCommitShort() {
        return steps.sh(
            script: 'git rev-parse --short HEAD',
            returnStdout: true
        ).trim()
    }
    
    def getVersion() {
        def version = steps.sh(
            script: 'cat version.txt',
            returnStdout: true
        ).trim()
        return version ?: '1.0.0'
    }
    
    def sendEmail(String to, String subject, String body) {
        steps.emailext(
            to: to,
            subject: subject,
            body: body
        )
    }
}
EOF

# 7. Create template Dockerfile
cat > resources/Dockerfile.template << 'EOF'
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
EOF

# 8. Initialize git repository
git init
git add .
git commit -m "Initial shared library"

# 9. Push to remote
git remote add origin https://github.com/your-org/jenkins-shared-library.git
git push -u origin main

# 10. Configure in Jenkins
# Go to Manage Jenkins > Configure System > Global Pipeline Libraries
# Add library with name 'my-shared-library' and point to repository
```

**Using Shared Library:**
```groovy
@Library('my-shared-library') _

pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                script {
                    def utils = new org.company.Utils(this)
                    def version = utils.getVersion()
                    def commit = utils.getGitCommitShort()
                    
                    buildDockerImage(
                        imageName: 'myapp',
                        imageTag: "${version}-${commit}"
                    )
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    deployToKubernetes(
                        deployment: 'myapp',
                        namespace: 'production',
                        image: "myapp:${version}-${commit}"
                    )
                }
            }
        }
    }
    
    post {
        success {
            notifySlack(
                message: 'Deployment successful!',
                color: 'good'
            )
        }
        failure {
            notifySlack(
                message: 'Deployment failed!',
                color: 'danger'
            )
        }
    }
}
```

**Challenge:**
Create shared library with:
1. Database migration functions
2. Terraform deployment
3. Security scanning
4. Performance testing

---

### Exercise 4: Multi-Branch Pipeline (45 minutes)

**Objective:** Set up multi-branch pipeline with different strategies

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        APP_NAME = 'myapp'
        DOCKER_REGISTRY = 'registry.example.com'
    }
    
    stages {
        stage('Determine Strategy') {
            steps {
                script {
                    // Different strategies for different branches
                    if (env.BRANCH_NAME == 'main') {
                        env.DEPLOY_ENV = 'production'
                        env.RUN_TESTS = 'true'
                        env.REQUIRE_APPROVAL = 'true'
                    } else if (env.BRANCH_NAME == 'develop') {
                        env.DEPLOY_ENV = 'staging'
                        env.RUN_TESTS = 'true'
                        env.REQUIRE_APPROVAL = 'false'
                    } else if (env.BRANCH_NAME.startsWith('feature/')) {
                        env.DEPLOY_ENV = 'dev'
                        env.RUN_TESTS = 'true'
                        env.REQUIRE_APPROVAL = 'false'
                    } else {
                        env.DEPLOY_ENV = 'none'
                        env.RUN_TESTS = 'true'
                        env.REQUIRE_APPROVAL = 'false'
                    }
                    
                    echo "Branch: ${env.BRANCH_NAME}"
                    echo "Deploy to: ${env.DEPLOY_ENV}"
                }
            }
        }
        
        stage('Build') {
            steps {
                sh """
                    docker build \
                      -t ${DOCKER_REGISTRY}/${APP_NAME}:${BRANCH_NAME}-${BUILD_NUMBER} \
                      .
                """
            }
        }
        
        stage('Test') {
            when {
                expression { env.RUN_TESTS == 'true' }
            }
            steps {
                sh 'npm test'
            }
        }
        
        stage('Approval') {
            when {
                expression { env.REQUIRE_APPROVAL == 'true' }
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
            }
        }
        
        stage('Deploy') {
            when {
                expression { env.DEPLOY_ENV != 'none' }
            }
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} \
                      ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BRANCH_NAME}-${BUILD_NUMBER} \
                      -n ${DEPLOY_ENV}
                """
            }
        }
    }
}
```

**Challenge:**
Implement:
1. PR validation pipeline
2. Hotfix workflow
3. Release pipeline
4. Canary deployment

---

### Exercise 5: Jenkins Configuration as Code (45 minutes)

**Objective:** Manage Jenkins configuration with JCasC

```yaml
# jenkins.yaml
jenkins:
  systemMessage: "Jenkins configured with JCasC"
  numExecutors: 5
  mode: NORMAL
  scmCheckoutRetryCount: 3
  
  securityRealm:
    local:
      allowsSignup: false
      users:
        - id: "admin"
          password: "${ADMIN_PASSWORD}"
        - id: "developer"
          password: "${DEVELOPER_PASSWORD}"
  
  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            description: "Jenkins administrators"
            permissions:
              - "Overall/Administer"
            assignments:
              - "admin"
          - name: "developer"
            description: "Jenkins developers"
            permissions:
              - "Overall/Read"
              - "Job/Build"
              - "Job/Read"
              - "Job/Workspace"
            assignments:
              - "developer"

  clouds:
    - kubernetes:
        name: "kubernetes"
        serverUrl: "https://kubernetes.default"
        namespace: "jenkins"
        jenkinsUrl: "http://jenkins:8080"
        jenkinsTunnel: "jenkins-agent:50000"
        containerCapStr: "10"
        templates:
          - name: "maven"
            label: "maven"
            containers:
              - name: "maven"
                image: "maven:3.8-jdk-11"
                command: "cat"
                ttyEnabled: true

credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              scope: GLOBAL
              id: "github-credentials"
              username: "${GITHUB_USERNAME}"
              password: "${GITHUB_TOKEN}"
              description: "GitHub credentials"
          
          - string:
              scope: GLOBAL
              id: "slack-token"
              secret: "${SLACK_TOKEN}"
              description: "Slack token"
          
          - file:
              scope: GLOBAL
              id: "kubeconfig"
              fileName: "kubeconfig"
              secretBytes: "${KUBECONFIG_BASE64}"

tool:
  git:
    installations:
      - name: "Default"
        home: "git"
  
  maven:
    installations:
      - name: "Maven 3.8"
        properties:
          - installSource:
              installers:
                - maven:
                    id: "3.8.6"

unclassified:
  location:
    url: "https://jenkins.example.com/"
    adminAddress: "admin@example.com"
  
  slackNotifier:
    teamDomain: "myteam"
    tokenCredentialId: "slack-token"
    room: "#builds"
  
  globalLibraries:
    libraries:
      - name: "my-shared-library"
        defaultVersion: "main"
        retriever:
          modernSCM:
            scm:
              git:
                remote: "https://github.com/org/jenkins-shared-library.git"
                credentialsId: "github-credentials"

jobs:
  - script: >
      multibranchPipelineJob('my-app') {
        branchSources {
          git {
            remote('https://github.com/org/my-app.git')
            credentialsId('github-credentials')
          }
        }
        orphanedItemStrategy {
          discardOldItems {
            numToKeep(20)
          }
        }
      }
```

**Challenge:**
Configure:
1. Multiple job templates
2. Plugin configuration
3. System groovy scripts
4. Backup configuration

---

## 🎯 Practice Projects

### Project 1: Complete CI/CD Pipeline
Build end-to-end pipeline:
- Multi-stage build
- Automated testing
- Security scanning
- Container registry
- Kubernetes deployment
- Monitoring integration
- Rollback capability

### Project 2: Multi-Environment Deployment
Implement deployment strategy:
- Dev/Staging/Prod environments
- Environment-specific configuration
- Approval gates
- Blue-green deployment
- Canary releases
- Automated rollback

### Project 3: Jenkins as Code
Fully automated Jenkins:
- Configuration as Code
- Job DSL for job creation
- Shared libraries
- Plugin management
- Backup and restore
- Disaster recovery

### Project 4: Distributed Build System
Scale Jenkins:
- Master-agent architecture
- Cloud agents (AWS/Azure/GCP)
- Kubernetes agents
- Auto-scaling
- Load balancing
- Monitoring

---

## 📝 Best Practices

### 1. Pipeline Design
- Use declarative pipelines
- Keep pipelines simple
- Use shared libraries
- Implement proper error handling
- Add meaningful stage names
- Use parallel execution
- Clean workspace after build

### 2. Security
- Use credentials plugin
- Implement RBAC
- Enable audit logging
- Scan for vulnerabilities
- Use least privilege
- Rotate credentials regularly
- Enable HTTPS

### 3. Performance
- Use agents efficiently
- Implement caching
- Optimize Docker builds
- Clean old builds
- Monitor resource usage
- Use pipeline caching
- Limit concurrent builds

### 4. Maintenance
- Regular backups
- Update plugins
- Monitor disk space
- Review logs
- Clean old workspaces
- Document pipelines
- Version control configuration

---

## ✅ Completion Checklist

### Day 1
- [ ] Install Jenkins
- [ ] Configure basic settings
- [ ] Create first job
- [ ] Set up build triggers
- [ ] Understand workspace

### Day 2
- [ ] Create declarative pipeline
- [ ] Use Jenkinsfile
- [ ] Add multiple stages
- [ ] Configure environment variables
- [ ] Implement post actions

### Day 3
- [ ] Use parallel execution
- [ ] Implement matrix builds
- [ ] Create shared library
- [ ] Use pipeline templates
- [ ] Handle errors properly

### Day 4
- [ ] Integrate with Git
- [ ] Use Docker in pipeline
- [ ] Deploy to Kubernetes
- [ ] Configure notifications
- [ ] Add code quality checks

### Day 5
- [ ] Configure authentication
- [ ] Implement RBAC
- [ ] Manage credentials
- [ ] Enable security features
- [ ] Set up audit logging

### Day 6
- [ ] Configure agents
- [ ] Set up cloud agents
- [ ] Use Docker agents
- [ ] Configure Kubernetes agents
- [ ] Implement auto-scaling

### Day 7
- [ ] Monitor Jenkins
- [ ] Optimize performance
- [ ] Implement backups
- [ ] Set up high availability
- [ ] Troubleshoot issues

---

**Next:** [Docker →](../docker/README.md)