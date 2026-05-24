# Project Concept: Kubernetes-Based DevOps Lifecycle for Spring PetClinic

**Author:** Moetez Cherni
**Registration Number:** 109463
**Course:** DevOps
**Application repository:** [https://github.com/ChMoetaz/spring-petclinic](https://github.com/ChMoetaz/spring-petclinic)
**Infrastructure repository:** [https://github.com/ChMoetaz/petclinic-devops-infra](https://github.com/ChMoetaz/petclinic-devops-infra)

---

## 1. Project Goal

The goal of this project is to implement the software lifecycle of an existing Spring Boot application in a reproducible, locally hosted Kubernetes environment.

The application used for this project is **Spring PetClinic**, a sample web application for managing a veterinary clinic. The application includes owners, pets, veterinarians and visits, and it requires a database as a backing service. The focus of this project is not to build a new application from scratch, but to create a working DevOps setup around an existing deployable workload.

The lifecycle will include building the application, testing it, publishing a container image, deploying it into different target environments, exposing the services through HTTPS and observing the running system through monitoring.

The setup is intentionally kept local and reproducible. Instead of using a large managed cloud setup, the project uses a local Kubernetes cluster as a cloud-simulated environment. This keeps the scope realistic while still allowing the implementation of Kubernetes concepts such as deployments, services, namespaces, ingress, rolling updates and horizontal scaling.

---

## 2. Repository Model

The project uses two repositories.

### 2.1 Application Repository

The first repository is a fork of the original Spring PetClinic application:

[https://github.com/ChMoetaz/spring-petclinic](https://github.com/ChMoetaz/spring-petclinic)

The application is not copied manually into a new repository. It is forked from the upstream Spring PetClinic repository so that the original origin and commit history remain visible.

This repository contains:

* Spring PetClinic source code
* application-specific changes made for this project
* Dockerfile for building the application image
* GitHub Actions workflow for build, test and image publishing if kept close to the application code
* README with onboarding information

Only small application-level changes are planned. The business logic of PetClinic does not need to be rewritten. A small visible change may be introduced during the review to demonstrate a complete lifecycle iteration from source code change to production deployment.

### 2.2 Infrastructure Repository

The second repository contains the infrastructure code, deployment configuration, documentation and this concept:

[https://github.com/ChMoetaz/petclinic-devops-infra](https://github.com/ChMoetaz/petclinic-devops-infra)

This repository contains:

* `concept.md`
* `README.md` with onboarding section
* `Makefile`
* helper scripts in `scripts/`
* Helm chart for Spring PetClinic and MySQL
* Kubernetes manifests for namespaces, ingress and TLS secrets if required
* Prometheus and Grafana configuration
* CI/CD workflow configuration if deployment automation is kept in the infrastructure repository
* documentation of relevant terminal commands

The infrastructure repository is the main place where the reproducibility of the overall setup is documented and automated.

---

## 3. Technology Stack

| Component             | Technology                               | Reason                                                                              |
| --------------------- | ---------------------------------------- | ----------------------------------------------------------------------------------- |
| Application           | Spring PetClinic / Spring Boot           | Existing open-source web application with database support                          |
| Build tool            | Maven Wrapper                            | Already part of the application and reproducible                                    |
| Version control       | GitHub                                   | Stores the application fork and the infrastructure repository                       |
| CI/CD                 | GitHub Actions with a self-hosted runner | Pipeline is triggered by VCS changes and can deploy to the local Kubernetes cluster |
| Artifact registry     | GitHub Container Registry                | Container images are explicitly published and versioned                             |
| Runtime environment   | Minikube Kubernetes cluster              | Local cloud-simulated environment with Kubernetes features                          |
| Deployment automation | Helm                                     | Reproducible application deployment and environment-specific values                 |
| Backing service       | MySQL                                    | Database required by the application                                                |
| Ingress / vhosting    | NGINX Ingress Controller                 | Exposes application and monitoring services through FQDNs                           |
| TLS                   | mkcert certificates                      | Provides HTTPS for local services without requiring a public domain                 |
| Monitoring            | Prometheus + Grafana                     | Monitoring stack provisioned as part of the project                                 |
| Logging               | Kubernetes container logs                | Application writes logs to stdout/stderr, inspectable through Kubernetes            |
| Local automation      | Makefile and shell scripts               | Makes setup, teardown and deployment commands transparent and repeatable            |

The selected tools are used because they fit this project setup and directly support the assignment requirements: reproducibility, automation, published artifacts, HTTPS access, monitoring, multiple environments, redundancy and zero-downtime deployment. They are not presented as universal best practices.

---

## 4. Target Environments

The project provides two target environments inside the Kubernetes cluster.

| Environment  | Kubernetes namespace   | Purpose                                                            | Deployment trigger            |
| ------------ | ---------------------- | ------------------------------------------------------------------ | ----------------------------- |
| `staging`    | `petclinic-staging`    | Non-production environment used to validate a new version          | Push or merge to `main`       |
| `production` | `petclinic-production` | Production-like environment used for the final reviewed deployment | Release tag + manual approval |

The environments are separated by Kubernetes namespaces, Helm values, services and database instances. This means staging and production can run different configurations while still using the same deployment mechanism.

The production application will run with at least two replicas. Production deployment requires a manual approval step in GitHub Actions to avoid accidental releases.

---

## 5. Infrastructure Architecture

The infrastructure is hosted on a local workstation or local Linux VM using Minikube. No project service is installed directly on the host system unless it is a required local tool such as Docker, Minikube, kubectl, Helm, mkcert or the GitHub Actions self-hosted runner.

All project services run inside Kubernetes:

* Spring PetClinic
* MySQL
* NGINX Ingress Controller
* Prometheus
* Grafana

### Architecture Diagram

```text
Developer
   |
   | git push / git tag
   v
GitHub repositories
   |
   | triggers workflow
   v
GitHub Actions
self-hosted runner on local machine
   |
   | build, test, docker build, docker push, helm upgrade
   v
GitHub Container Registry
   |
   | Kubernetes pulls image
   v
Minikube Kubernetes Cluster
   |
   |-- namespace: petclinic-staging
   |     |-- Spring PetClinic Deployment
   |     |-- MySQL Deployment
   |     |-- Service
   |     |-- Ingress: staging.petclinic.local
   |
   |-- namespace: petclinic-production
   |     |-- Spring PetClinic Deployment, replicas >= 2
   |     |-- MySQL Deployment
   |     |-- Service
   |     |-- Ingress: petclinic.local
   |
   |-- namespace: monitoring
         |-- Prometheus
         |-- Grafana
         |-- Ingress: prometheus.petclinic.local
         |-- Ingress: grafana.petclinic.local
```

The self-hosted GitHub Actions runner runs outside Kubernetes on the local workstation and deploys to Minikube through `kubectl` and `helm`. The application itself runs inside Kubernetes. This separates the pipeline execution process from the deployed application runtime.

---

## 6. Infrastructure as Code and Local Workflow

The infrastructure will be described through code and reproducible commands. The main workflow will be implemented with a `Makefile` and scripts first. The CI/CD pipeline can then call the same commands later.

Example commands:

```bash
make cluster-start
make install-ingress
make install-monitoring
make deploy-staging
make deploy-production
make status
make destroy
```

This approach keeps the setup understandable and testable during development. It also makes the review easier because every relevant terminal command is visible in the repository.

The setup will be destroyed and recreated regularly during development to check reproducibility. A reviewer should be able to follow the README and recreate the complete environment from the repository content.

---

## 7. Application Lifecycle and Pipeline

At least one CI/CD pipeline will put the application through the required stages: build, test and deploy.

### 7.1 Build Stage

A change in the VCS triggers the pipeline. The pipeline checks out the application source code and builds the project using the Maven Wrapper.

Tasks:

* checkout repository
* run Maven build
* build Docker image
* tag image with commit SHA

### 7.2 Test Stage

The pipeline executes the available automated tests.

Tasks:

* run unit and integration tests
* fail the pipeline if tests fail
* keep the application in a deployable state

### 7.3 Publish Stage

After a successful build and test stage, the Docker image is pushed to GitHub Container Registry.

The container image is the explicit artifact of this project. It will be tagged with the commit SHA and, for production releases, with a release tag.

### 7.4 Deploy to Staging

A push or merge to the `main` branch deploys the image to the `petclinic-staging` namespace using Helm.

Example command executed by the pipeline:

```bash
helm upgrade --install petclinic ./helm/petclinic \
  --namespace petclinic-staging \
  --values ./helm/petclinic/values-staging.yaml \
  --set image.tag=$IMAGE_TAG
```

### 7.5 Deploy to Production

Production deployment is controlled. A release tag, for example `v1.0.0`, triggers the production workflow. The deployment still requires a manual approval step before execution.

Example command:

```bash
helm upgrade --install petclinic ./helm/petclinic \
  --namespace petclinic-production \
  --values ./helm/petclinic/values-production.yaml \
  --set image.tag=$IMAGE_TAG
```

This makes the difference between staging and production explicit in the lifecycle automation.

---

## 8. Deployment Strategy, Redundancy and Zero Downtime

The production deployment will run with at least two replicas.

Example configuration:

```yaml
replicaCount: 2
```

Kubernetes rolling updates will be used to keep the application reachable during deployment.

Example deployment strategy:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

Readiness probes will be configured so that Kubernetes only routes traffic to pods that are ready.

Example:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 8080
```

The exact probe endpoint may be adjusted depending on the final application configuration.

This setup supports zero-downtime deployment because new pods must become ready before old pods are removed from service.

---

## 9. Backing Service and Persistence Layer

Spring PetClinic requires a database. MySQL will be deployed inside Kubernetes as the backing service.

For this project, the persistence layer may be considered ephemeral. Data loss after restarting or recreating the database service is acceptable for the assignment. The important part is that the database is part of the infrastructure code and that the application is configured to use it as an attached backing service.

Each environment will have its own MySQL deployment and service.

Application configuration such as database URL, username, password and active Spring profile will be provided through Kubernetes configuration objects and secrets, not hardcoded into the application source code.

---

## 10. FQDN, HTTPS and Ingress

All relevant services will be accessible through FQDNs and HTTPS.

Planned local FQDNs:

| Service                | FQDN                         |
| ---------------------- | ---------------------------- |
| Staging application    | `staging.petclinic.local`    |
| Production application | `petclinic.local`            |
| Grafana                | `grafana.petclinic.local`    |
| Prometheus             | `prometheus.petclinic.local` |

For local DNS simulation, these names can be mapped to the Minikube ingress IP through the host system's hosts file.

Example:

```text
<minikube-ip> petclinic.local
<minikube-ip> staging.petclinic.local
<minikube-ip> grafana.petclinic.local
<minikube-ip> prometheus.petclinic.local
```

TLS termination happens at the NGINX Ingress layer. Local certificates will be generated using mkcert and stored as Kubernetes TLS secrets.

---

## 11. Monitoring and Logging

Monitoring is implemented using Prometheus and Grafana. Both are provisioned inside the Kubernetes cluster and are part of the infrastructure code.

Prometheus will collect metrics from Kubernetes and available service endpoints. Grafana will visualize these metrics through dashboards.

The following aspects will be monitored:

* pod availability
* pod restarts
* CPU usage
* memory usage
* deployment status
* application service availability
* Kubernetes cluster health where available

Logging is kept simple. The application writes logs to stdout/stderr. Logs can be inspected with Kubernetes commands.

Example:

```bash
kubectl logs deployment/petclinic -n petclinic-production
```

A larger logging stack such as EFK or Loki is intentionally not part of the initial scope to keep the project manageable.

---

## 12. 12-Factor Considerations

The application and deployment will follow relevant 12-factor ideas where reasonable for this project.

| Factor              | Application in this project                                                                      |
| ------------------- | ------------------------------------------------------------------------------------------------ |
| Codebase            | The application is kept in one forked repository with visible upstream origin                    |
| Dependencies        | Application dependencies are declared through Maven                                              |
| Config              | Environment-specific configuration is injected through Kubernetes values, ConfigMaps and Secrets |
| Backing services    | MySQL is treated as an attached backing service                                                  |
| Build, release, run | Build/test, image publishing and deployment are separated by the pipeline                        |
| Processes           | Application containers are treated as stateless; database state is external to the app container |
| Port binding        | Spring Boot exposes the application on a configured container port                               |
| Concurrency         | Production uses multiple replicas of the application                                             |
| Disposability       | Kubernetes can replace pods and roll out new versions                                            |
| Dev/prod parity     | Staging and production use the same container image and Helm chart with different values         |
| Logs                | Application logs are written to stdout/stderr                                                    |

---

## 13. Review Demonstration Plan

During the review, the following lifecycle iteration can be demonstrated:

1. Clone the infrastructure repository.
2. Start or verify the local Kubernetes cluster.
3. Install ingress and monitoring using Makefile commands.
4. Deploy staging and production environments.
5. Verify that application, Grafana and Prometheus are reachable through FQDNs and HTTPS.
6. Introduce a small visible change in the application repository.
7. Commit and push the change.
8. Show that the CI pipeline is triggered by the VCS change.
9. Show build and test stages.
10. Show the published container image in GitHub Container Registry.
11. Deploy the new image to staging.
12. Approve or trigger production deployment.
13. Verify that production remains reachable during rollout.
14. Check Kubernetes rollout status and monitoring dashboards.

---

## 14. Scope Limitation

The project intentionally avoids a managed public cloud setup such as GKE, EKS or AKS. It also avoids additional services such as Cloud SQL, external secret managers or a full logging stack.

The reason is to keep the implementation realistic and reproducible within the course timeframe while still meeting the technical requirements.

The focus remains on:

* version control
* build and test automation
* artifact publishing
* Kubernetes deployment
* Infrastructure as Code
* staging and production environments
* manual production approval
* redundancy
* zero-downtime deployment
* FQDN and HTTPS access
* monitoring
* reproducibility from scratch

---

## 15. Indication of Source

All external sources and AI-assisted changes will be indicated according to the course requirements.

For static sources, deep links or bibliographic references will be placed close to the affected content.

For AI-assisted text or code, the related changes will be committed separately. Commit messages will use the required `ai:` prefix and include a human-readable excerpt of the prompt history that led to the affected change.

Example commit title:

```text
ai(chatgpt): revise Kubernetes project concept after assignment feedback
```

The purpose is to make transparent which parts of the concept or implementation were influenced by AI-based interaction.
