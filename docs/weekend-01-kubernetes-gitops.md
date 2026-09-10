# Weekend 01 — Kubernetes, Helm, CI and GitOps

## Objective

Take the existing FinOps AI workload and learn the production delivery path without changing the original application repository.

The focus is not merely getting YAML to work. At every step, understand what problem the technology solves and be able to explain the design in an interview.

## Target architecture

```text
                       Local Kubernetes

                         finops namespace
                               |
          +--------------------+--------------------+
          |                    |                    |
       portal              ai-platform          enterprise
    Deployment             Deployment           Deployment
       Service                Service              Service
                               |
                               v
                            Postgres
                         StatefulSet/PVC

GitHub Actions -> validate/test/chart lint
Git desired state -> Argo CD -> Kubernetes reconciliation
```

## Prerequisites

Install Docker, kubectl, Helm, kind and the Argo CD CLI. Confirm each command works before starting.

```bash
docker version
kubectl version --client
helm version
kind version
argocd version --client
```

## Lab 1 — Create a local Kubernetes cluster

```bash
kind create cluster --name finops-lab
kubectl cluster-info --context kind-finops-lab
kubectl create namespace finops
kubectl get namespaces
```

### What to understand

A Kubernetes cluster runs workloads. A namespace gives the FinOps workload a logical boundary. A Pod is the smallest schedulable unit, while a Deployment manages replica lifecycle and rolling updates.

### Interview check

**Q: What happens when a Pod created by a Deployment crashes?**

The Deployment's ReplicaSet observes that actual state no longer matches desired state and creates a replacement Pod. This reconciliation model is central to Kubernetes.

## Lab 2 — Deploy a first workload

Start with a simple nginx Deployment so the Kubernetes mechanics are clear before introducing the full FinOps application.

```bash
kubectl create deployment platform-demo --image=nginx:alpine -n finops
kubectl expose deployment platform-demo --port=80 -n finops
kubectl get pods -n finops
kubectl get deployments -n finops
kubectl get services -n finops
```

Scale it:

```bash
kubectl scale deployment platform-demo --replicas=3 -n finops
kubectl get pods -n finops
```

Delete one Pod and observe reconciliation:

```bash
kubectl get pods -n finops
kubectl delete pod <pod-name> -n finops
kubectl get pods -n finops -w
```

## Lab 3 — FinOps Kubernetes resources

Build manifests for these components:

```text
portal
ai-platform
enterprise
postgres
```

For each stateless application create a Deployment and Service. Add readiness/liveness probes where the application exposes health endpoints. Externalize non-secret configuration using ConfigMaps and sensitive values using Kubernetes Secrets.

For learning purposes PostgreSQL can run in the cluster using a StatefulSet and persistent volume. In the later AWS lab it will move to RDS rather than remaining inside EKS.

### Concepts to practice

- Deployment vs StatefulSet
- ClusterIP vs LoadBalancer
- ConfigMap vs Secret
- readiness vs liveness probes
- CPU/memory requests and limits
- rolling updates
- namespaces
- labels and selectors

## Lab 4 — Package with Helm

Create:

```text
helm/finops-platform/
  Chart.yaml
  values.yaml
  values-dev.yaml
  templates/
```

Initialize the chart if desired:

```bash
helm create helm/finops-platform
```

Useful commands:

```bash
helm lint helm/finops-platform
helm template finops helm/finops-platform -n finops
helm upgrade --install finops helm/finops-platform -n finops
helm list -n finops
```

### Interview check

**Q: Why Helm when Kubernetes already supports YAML?**

Kubernetes YAML describes resources. Helm provides reusable packaging and parameterization, allowing the same application definition to be deployed with environment-specific values rather than copying manifests for every environment.

## Lab 5 — GitHub Actions CI

The CI pipeline for this lab should initially validate the platform configuration rather than deploy it.

Desired flow:

```text
Pull request
    |
    +--> YAML validation
    +--> helm lint
    +--> helm template
    +--> optional application tests
```

Later the workflow will build images and push them to AWS ECR.

### Interview check

**Q: Is GitHub Actions CI or CD here?**

Primarily CI. It validates changes and eventually builds/publishes immutable artifacts. Argo CD owns deployment reconciliation.

## Lab 6 — Argo CD

Install Argo CD into the local cluster and create an Application pointing to the Helm chart in this repository.

Conceptual flow:

```text
Git repository
      |
      v
   Argo CD
      |
 compare desired vs actual
      |
      v
 Kubernetes
```

Make a controlled change such as increasing a replica count in Git. Observe Argo CD detect the difference and synchronize it to Kubernetes.

Then manually modify the cluster and observe drift.

### Interview check

**Q: Why use Argo CD instead of kubectl from GitHub Actions?**

Argo CD continuously reconciles Kubernetes against Git. Git becomes the auditable desired state, deployment credentials do not need to live in the CI pipeline, and drift can be detected and corrected.

## Weekend completion test

You are finished when you can demonstrate and explain:

1. A local Kubernetes cluster running multiple services.
2. A Pod being deleted and automatically recreated.
3. A Deployment scaled from one replica to three.
4. A failed readiness probe preventing traffic from reaching a Pod.
5. The application packaged as a Helm chart.
6. GitHub Actions validating the chart.
7. Argo CD detecting and reconciling a Git change.
8. Argo CD detecting manual cluster drift.

## Interview story

Use this structure rather than memorizing definitions:

> I started with a multi-runtime FinOps AI application that ran through Docker Compose. I created a separate platform-engineering repository and moved the workload model into Kubernetes. Stateless components run as Deployments and Services, configuration is externalized, health probes control traffic and recovery, and Helm packages the deployment for different environments. GitHub Actions validates and builds artifacts while Argo CD handles GitOps deployment and continuous reconciliation. The same model can later be promoted from local kind to AWS EKS using Terraform-managed infrastructure.

## Next weekend

Weekend 02 adds Kafka for settlement-domain events, Temporal for durable human-in-the-loop investigation workflows, and a small Go gRPC service to demonstrate strongly typed internal service communication.
