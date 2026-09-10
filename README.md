# FinOps Platform Engineering Lab

A hands-on platform engineering project built around the existing **FinOps AI — Agentic Trade & Settlement Operations Platform**.

The original application remains unchanged. This repository demonstrates how to operate that workload using cloud-native platform engineering practices.

## Source application

- Application repo: `acherukuri83-stack/finops-ai-final-c`
- Application stack: React/TypeScript, Python/FastAPI, Java/Spring Boot, PostgreSQL/pgvector, OpenTelemetry

## What this lab demonstrates

- Kubernetes
- Helm
- GitHub Actions
- Argo CD / GitOps
- AWS / EKS / ECR / RDS
- Terraform
- Kafka
- Temporal
- Go
- gRPC & Protobuf
- Datadog
- Snowflake
- Tilt
- Internal Platform APIs and developer tooling

## Learning path

### Weekend 1 — Kubernetes delivery foundation
1. Run the FinOps workload on local Kubernetes.
2. Understand Pods, Deployments, Services, ConfigMaps, Secrets, probes and resource limits.
3. Package Kubernetes resources with Helm.
4. Validate charts in GitHub Actions.
5. Deploy the Helm release through Argo CD.

### Weekend 2 — Distributed systems
1. Publish settlement-domain events through Kafka.
2. Create a durable investigation/approval workflow with Temporal.
3. Add a small Go service using gRPC and Protobuf.

### Weekend 3 — AWS and Infrastructure as Code
1. Provision VPC, EKS, ECR and RDS with Terraform.
2. Deploy application workloads to EKS.
3. Introduce IAM/workload identity and environment separation.

### Weekend 4 — Production platform concerns
1. Send OpenTelemetry data to Datadog.
2. Export analytical data to Snowflake.
3. Use Tilt for the local Kubernetes development loop.
4. Add internal platform APIs / CLI workflows.

## Repository layout

```text
kubernetes/       Raw Kubernetes learning manifests
helm/             Reusable Helm chart
argocd/           GitOps application definitions
terraform/        AWS infrastructure as code
kafka/            Event-driven architecture exercises
temporal/         Durable workflow exercises
grpc-services/    Go + gRPC + Protobuf exercises
observability/    Datadog/OpenTelemetry configuration
snowflake/        Analytics exercises
tilt/             Local developer workflow
docs/             Weekend labs, architecture and interview notes
```

## Weekend 1 goal

By the end of the first weekend you should be able to explain this delivery path in an interview:

```text
Developer -> GitHub -> GitHub Actions -> container images
                                      |
                                      v
                                Git desired state
                                      |
                                      v
                                   Argo CD
                                      |
                                      v
                                Kubernetes
                                      |
                                      v
                              Helm-managed apps
```

Start with [`docs/weekend-01-kubernetes-gitops.md`](docs/weekend-01-kubernetes-gitops.md).
