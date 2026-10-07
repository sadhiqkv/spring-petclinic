# Spring PetClinic – DevOps CI/CD Project

A complete DevOps CI/CD project for deploying the Spring PetClinic application using Jenkins, Maven, Trivy, SonarQube, Nexus, Docker, Docker Hub and Kubernetes (Minikube), with Prometheus, Grafana and Blackbox Exporter for monitoring.

---

## Project Overview

This project demonstrates an end-to-end CI/CD pipeline for a Java Spring Boot application.

The pipeline automates:

* Source code checkout from GitHub
* Maven build and testing
* Trivy filesystem/dependency security scanning
* SonarQube code quality analysis
* SonarQube Quality Gate validation
* Maven artifact deployment to Nexus
* Docker image creation
* Trivy Docker image security scanning
* Docker Hub image publishing
* Kubernetes deployment on Minikube
* Application and Kubernetes monitoring using Prometheus and Grafana
* Endpoint availability monitoring using Blackbox Exporter

---

## CI/CD Pipeline Flow

```text
Developer
    |
    v
  GitHub
    |
    v
  Jenkins
    |
    +--> Maven Build & Test
    |
    +--> Trivy Filesystem Scan
    |
    +--> SonarQube Analysis
    |
    +--> Quality Gate
    |
    +--> Nexus Repository
    |
    +--> Docker Image Build
    |
    +--> Trivy Docker Image Scan
    |
    +--> Docker Hub
    |
    +--> Minikube / Kubernetes
    |
    v
PetClinic Application
    |
    v
Monitoring
    |
    +--> Prometheus
    |
    +--> Grafana
    |
    +--> Blackbox Exporter
```

---

## Tools and Technologies

| Tool              | Purpose                                 |
| ----------------- | --------------------------------------- |
| GitHub            | Source code management                  |
| Jenkins           | CI/CD automation                        |
| Maven             | Java build and testing                  |
| Trivy             | Security and vulnerability scanning     |
| SonarQube         | Static code quality analysis            |
| Quality Gate      | Quality pass/fail checkpoint            |
| Nexus             | Maven artifact repository               |
| Docker            | Application containerization            |
| Docker Hub        | Container image registry                |
| Kubernetes        | Container orchestration                 |
| Minikube          | Local/lab Kubernetes cluster            |
| Prometheus        | Metrics collection and monitoring       |
| Grafana           | Monitoring dashboards and visualization |
| Blackbox Exporter | Endpoint availability monitoring        |

---

## Architecture

The project uses separate EC2 servers for the major DevOps components.

```text
                    GitHub
                       |
                       v
                    Jenkins
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
     Trivy         SonarQube        Maven
        |              |
        |        Quality Gate
        |              |
        +--------------+
                       |
                       v
                     Nexus
                       |
                       v
                 Docker Build
                       |
                       v
                 Trivy Image Scan
                       |
                       v
                   Docker Hub
                       |
                       v
              Minikube / Kubernetes
                       |
                +------+------+
                |             |
                v             v
            PetClinic       PostgreSQL
                |
                v
          Monitoring Server
                |
       +--------+---------+
       |        |         |
       v        v         v
 Prometheus  Grafana  Blackbox
```

---

## CI/CD Stages

### 1. Checkout

Jenkins checks out the application source code from the GitHub repository.

### 2. Maven Build & Test

Maven compiles the application, runs the tests and generates the Spring Boot JAR.

```bash
./mvnw clean verify
```

### 3. Trivy Filesystem Scan

Trivy scans the project and its dependencies for known vulnerabilities.

The current project uses the scan as a report-only security check.

```bash
trivy fs --severity HIGH,CRITICAL --exit-code 0 .
```

### 4. SonarQube Analysis

SonarQube analyzes the source code for:

* Bugs
* Vulnerabilities
* Code smells
* Reliability
* Maintainability
* Code coverage
* Duplications

### 5. Quality Gate

Jenkins waits for the SonarQube Quality Gate result.

```text
SonarQube
    |
    v
Quality Gate
    |
    +--> Passed → Continue
    |
    +--> Failed → Stop Pipeline
```

### 6. Nexus Deploy

After the Quality Gate passes, Maven deploys the generated JAR to the Nexus Maven repository.

Nexus provides centralized storage for build artifacts.

### 7. Docker Build

Jenkins creates a Docker image from the Spring Boot JAR.

Example:

```text
sadhiqkv/spring-petclinic:13
```

The Jenkins build number is used as the image tag.

### 8. Trivy Docker Image Scan

The generated Docker image is scanned for known vulnerabilities.

```bash
trivy image --severity HIGH,CRITICAL --exit-code 0 <image>
```

### 9. Docker Hub Push

The validated Docker image is pushed to Docker Hub.

Both the build-number tag and `latest` tag are published.

Example:

```text
sadhiqkv/spring-petclinic:13
sadhiqkv/spring-petclinic:latest
```

### 10. Kubernetes Deployment

Jenkins connects to the Minikube EC2 server through SSH and deploys the application.

The Kubernetes deployment contains:

* PetClinic application
* PostgreSQL database
* Kubernetes Services
* Deployment resources
* Health probes

Example verification:

```bash
minikube kubectl -- get pods
minikube kubectl -- get services
```

### 11. Monitoring

A separate monitoring EC2 server is used for:

**Prometheus**

* Collects metrics
* Stores time-series monitoring data

**Grafana**

* Displays Prometheus metrics through dashboards
* Helps visualize application and infrastructure health

**Blackbox Exporter**

* Performs external endpoint probes
* Can verify whether the PetClinic HTTP endpoint is reachable

---

## Kubernetes Components

### PetClinic

Spring Boot application running inside a Kubernetes Pod.

### PostgreSQL

Database used by the PetClinic application.

### Services

The database uses an internal ClusterIP Service.

PetClinic uses a NodePort Service for application access in the Minikube environment.

---

## Project Repository Structure

```text
spring-petclinic/
│
├── .mvn/
├── src/
│   ├── main/
│   └── test/
│
├── k8s/
│   ├── db.yml
│   └── petclinic.yml
│
├── Dockerfile
├── Jenkinsfile
├── docker-compose.yaml
├── pom.xml
├── mvnw
├── mvnw.cmd
├── .gitignore
└── README.md
```

Monitoring configuration can be maintained separately, for example:

```text
monitoring/
├── prometheus/
├── blackbox/
└── grafana/
```

---

## Important Jenkins Pipeline Stages

The Jenkins pipeline follows this order:

```text
Checkout
   ↓
Maven Build & Test
   ↓
Trivy Filesystem Scan
   ↓
SonarQube Analysis
   ↓
Quality Gate
   ↓
Nexus Deploy
   ↓
Docker Build
   ↓
Trivy Docker Image Scan
   ↓
Docker Hub Push
   ↓
Deploy to Minikube
   ↓
Kubernetes Verification
```

---

## Security Scanning

Trivy is used at two points:

```text
Source Code
    ↓
Trivy Filesystem Scan
    ↓
Build
    ↓
Docker Image
    ↓
Trivy Image Scan
```

The current project is configured to report HIGH and CRITICAL vulnerabilities without failing the demonstration pipeline.

For a production environment, security policies can be configured to fail the pipeline when vulnerabilities exceed an approved threshold.

---

## Troubleshooting

### Nexus 401 Unauthorized

**Problem:**

Maven deployment returned HTTP 401.

**Cause:**

Maven credentials were not supplied using the correct server configuration.

**Solution:**

Jenkins creates a temporary Maven `settings.xml` containing the Nexus credentials and uses the matching server ID during deployment.

---

### Checkstyle failed because of Nexus HTTP URL

**Problem:**

The Maven Checkstyle/NoHttp rule detected the HTTP Nexus URL.

**Cause:**

A temporary `nexus-settings.xml` file had remained in the Jenkins workspace.

**Solution:**

The leftover file was removed and the pipeline was updated to automatically delete the temporary settings file after the Nexus stage.

---

### Trivy disk quota error

**Problem:**

Trivy failed while downloading/using its vulnerability database.

**Cause:**

The available temporary filesystem space was insufficient.

**Solution:**

The EC2 root storage was increased and `/var/tmp/trivy` was used as the temporary directory.

---

### Maven tests failed because docker-compose.yaml was missing

**Problem:**

The test lifecycle expected Docker Compose configuration.

**Solution:**

`docker-compose.yaml` was added to the project repository and Docker Compose v2 was installed on Jenkins.

---

### Kubernetes ImagePullBackOff

**Problem:**

The PetClinic Pod could not pull the expected image because an old manually-created deployment contained incorrect container configuration.

**Solution:**

The old PetClinic deployment was removed and Jenkins recreated the deployment using the correct Kubernetes manifest and Docker Hub image.

---

### CrashLoopBackOff

**Meaning:**

The container starts, crashes, and Kubernetes repeatedly attempts to restart it.

Troubleshooting commands:

```bash
minikube kubectl -- logs <pod-name>
```

```bash
minikube kubectl -- describe pod <pod-name>
```

The logs and Events section are used to find the actual cause.

---

### Minikube NodePort not directly accessible through EC2 public IP

**Problem:**

The Minikube NodePort was not directly reachable through the EC2 public IP.

**Cause:**

Minikube was running with the Docker driver, so the Kubernetes node network existed inside the Minikube/Docker network.

**Solution for evaluation:**

An SSH local port-forward was used:

```bash
ssh -i "kube-key.pem" \
-L 8080:192.168.49.2:32645 \
ubuntu@<MINIKUBE_PUBLIC_IP>
```

The application could then be accessed through:

```text
http://localhost:8080
```

---

## Final Result

The project successfully demonstrates an end-to-end DevOps workflow:

```text
Code
 ↓
Build
 ↓
Security Scan
 ↓
Code Quality
 ↓
Quality Gate
 ↓
Artifact Repository
 ↓
Container Build
 ↓
Container Security Scan
 ↓
Container Registry
 ↓
Kubernetes Deployment
 ↓
Monitoring
```

The pipeline successfully deploys the Spring PetClinic application to Minikube and verifies the Kubernetes rollout.

---

## Screenshots

Screenshots from the project can be added to this section.

Suggested screenshots:

1. Jenkins successful pipeline
2. SonarQube dashboard
3. Nexus Maven repository
4. Github 
5. PetClinic running in browser
6. Prometheus targets
7. Grafana dashboard
8. DockerHub

Example:

```markdown
## Jenkins Pipeline

![Jenkins Pipeline](screenshots/jenkins-pipeline.png)
```

---

## Project Outcome

This project provided practical experience with:

* CI/CD automation
* Linux and AWS EC2
* Git and GitHub
* Jenkins
* Maven
* Trivy
* SonarQube
* Quality Gates
* Nexus Repository
* Docker
* Docker Hub
* Kubernetes
* Minikube
* Prometheus
* Grafana
* Blackbox Exporter

---

## Author

**Mohammad Sadhiq K.V.**

DevOps Trainee | Linux | AWS | Docker | Kubernetes | Jenkins | CI/CD
