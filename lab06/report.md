# Report Lab06 CICD for Java App

Author: Quentin Surdez

## Estimation

| Task                     | Estimated Time | Effective Time | Remarks                                                                      |
|--------------------------|----------------|----------------|------------------------------------------------------------------------------|
| Preparation              | 15min          | 5min           | Well, forking needs the user to be connected. I have learned something today |
| Dockerfile               | 20min          | 45min          | Switched from Gradle to Maven midway                                         |
| Docker Compose           | 30min          | 10min          | Simple configuration needed                                                  |
| Configure CI/CD          | 1h             | 3h30min        | Split into multiple pipeline files for better maintainability                |
| Configure K8s deployment | 2h             | 2h             | Used a standard deployment approach                                          |
| Report                   | 1h             | 1h             |                                                                              |
| Total                    | 5h05min        | 7h30min        |                                                                              |

## Build Tools Decision: Maven vs. Gradle

Initially, I experimented with Gradle for the build process but reverted to
Maven for several reasons:

- Gradle builds in CI/CD proved significantly slower than expected
- My deeper experience with Maven enabled more efficient pipeline configuration
- Maven's deterministic builds are easier to maintain in CI environments
- The switch required rewriting the Dockerfile and CI pipelines but provided
  long-term benefits in build consistency and speed

## Docker Implementation

### Dockerfile Analysis

The Dockerfile uses a multi-stage build approach to minimize the final image
size:

```dockerfile
FROM maven:3-eclipse-temurin-17 AS build
# Build stage operations
...

FROM debian:12.10-slim
# Runtime stage with only necessary components
...
```

Key features:

- Build stage: Uses maven:3-eclipse-temurin-17 to compile the application
- Runtime optimization: Uses jdeps and jlink to create a custom JRE with only
  required modules
- Minimal runtime image: Based on debian:12.10-slim with only the custom JRE
  and compiled JAR
- Security enhancement: Reduced attack surface with minimal dependencies

### Docker Compose Configuration

The Docker Compose file simplifies deployment with:

- Container naming for easier management
- Port mapping (8080:8080) for direct access
- Reference to the GitLab registry image
- Custom network creation for potential multi-container setups

## CI/CD Pipeline Architecture

The CI/CD implementation uses a parent-child pipeline structure with specialized
sub-pipelines:

### Parent Pipeline (gitlab-ci.yml)

- Implements conditional triggering based on file changes
- Optimizes resource usage by running only necessary pipelines

### App Pipeline (app-pipeline.yml)

- Serves as an orchestrator that triggers specialized child pipelines
- Handles building and testing the application
- Implements caching strategies for Maven dependencies
- Separates unit and integration tests
- Generates code coverage reports with JaCoCo
- Uses artifacts to share build outputs between stages

### Docker Pipeline (docker-pipeline.yml)

- Builds and publishes the Docker image
- Implements Docker layer caching for faster builds
- Includes container security scanning via GitLab template
- Publishes images only on the main branch

### Quality Pipeline (quality-pipeline.yml)

- Runs static code analysis tools (SpotBugs, PMD, Checkstyle)
- Implements dependency scanning via GitLab template
- Performs SAST (Static Application Security Testing)
- Produces quality reports as artifacts

### CI/CD Features Implementation

The pipeline incorporates several advanced CI/CD features:

#### Testing and Code Quality

- Unit Test Reports: JUnit XML reports are collected and displayed in GitLab
- Code Coverage: JaCoCo reports track code coverage percentage
- Static Analysis: Multiple tools analyze code quality and detect potential
  issues

#### Security Features

- SAST: Identifies code-level security vulnerabilities
- Dependency Scanning: Detects vulnerable dependencies in the application
- Container Scanning: Checks the Docker image for security issues

#### Optimization Techniques

- Caching: Maven repository is cached based on pom.xml changes
- Conditional Execution: Pipelines only run when relevant files change
- Artifact Management: Efficient sharing of build outputs between stages

All the different stages build artifacts and as we are free tier in Gitlab,
we cannot view them all. However, we can go to the Build -> Artifacts on our
project page and download the artifacts of interest.

## Kubernetes Implementation

The Kubernetes deployment adopts a comprehensive approach with multiple resource
types working together to ensure reliable, scalable operation of the Spring
application.

### Resource Organization

All resources are logically grouped within a dedicated namespace spring-app:

- Clear isolation from other applications in the cluster
- Easier resource management and monitoring

### Deployment Strategy

The deployment configuration implements several production-ready practices:

- Rolling Updates: Zero-downtime deployments with maxUnavailable: 0 setting
- Resource Management: Explicit CPU and memory requests and limits to prevent
  resource contention
- Health Monitoring: Both liveness and readiness probes to ensure application
  stability. It will need to be set to the actual path for the probes.
- Observability: Prometheus annotations for metrics collection
- Pod Scaling: Horizontal Pod Autoscaler that automatically scales based on
  CPU utilization

### Networking Configuration

The application is exposed through a layered network approach:

- ClusterIP Service: Internal access point for the application
- Ingress Resource: External HTTP routing to the service via the nginx
  ingress controller
- Named Ports: Clear identification of protocol and purpose (http/8080)

## How to test it locally ?

First be sure to have minikube and kubectl installed.

Then run:

```bash
minikube start
```

Make sure the ingress addon is enable:

```bash
minikube addons enable ingress
```

Deploy on your cluster:

```bash
kubectl apply -f deployment.yml
```

If you want to connect to the application you need to launch a tunnel from
minikube in a separate terminal window:

```bash
minikube tunnel
```

Then you will be able to connect to the address `localhost` and access the
application.

## Challenges and Solutions

- Build Tool Selection: Switched from Gradle to Maven for better CI/CD
  performance
- Pipeline Complexity: Split the monolithic pipeline into specialized
  sub-pipelines for better maintainability
- Docker Optimization: Implemented multi-stage builds and custom JRE to
  reduce image size
- CI/CD Performance: Added caching strategies and conditional execution to
  speed up pipelines
- Parent-Child Pipeline: This architecture is not very well supported for
  free tier account, so debugging it was quite difficule. Escpecially with
  the different rules that needed to be put into place so that everything is
  run on a MR but not necessarily on a push to a developing branch
