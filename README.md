# Nexus Platform

**Cloud-native DevSecOps & backend platform — secure CI/CD, Kubernetes orchestration, microservices architecture, and production-grade infrastructure.**

Built to demonstrate how modern systems are secured, deployed, and monitored across the full software delivery lifecycle — not just how they are coded.

---

## Why this project exists

Most portfolio projects show that someone can write code. This one is built to show something harder: that I understand how code gets to production securely, reliably, and at scale.

The gap between "I wrote a working app" and "I know how to ship it safely in a real organisation" is exactly what this platform is designed to close.

---

## System architecture

```
Client Requests
      │
      ▼
┌─────────────┐
│  API Gateway │  ← Authentication, rate limiting, routing
└──────┬──────┘
       │
  ┌────┴────┐
  │         │
  ▼         ▼
Service A  Service B  (independent microservices)
  │         │
  └────┬────┘
       │
  ┌────▼─────┐      ┌──────────────┐
  │ PostgreSQL│      │  Redis Cache │
  └──────────┘      └──────────────┘
       │
  ┌────▼──────────┐
  │  Kubernetes   │  ← Orchestration, scaling, RBAC
  └───────────────┘
       │
  ┌────▼──────────┐
  │ AWS (Terraform)│  ← Infrastructure as Code
  └───────────────┘
```

**Why microservices over a monolith?**
Each service can be scanned, deployed, and rolled back independently. In a security context this matters — a vulnerability in one service doesn't require redeploying everything, and blast radius is contained.

---

## Tech stack

### Backend
| Layer | Technology | Why |
|---|---|---|
| Runtime | Node.js | Fast prototyping, strong ecosystem for REST and middleware |
| API design | REST / Express | Straightforward for demonstrating input validation and auth flows |
| Database | PostgreSQL | Relational integrity for auth and user data |
| Cache | Redis | Session management and rate-limit counters |

### DevSecOps
| Tool | Purpose |
|---|---|
| Docker | Container image build and hardening |
| Kubernetes | Orchestration, horizontal scaling, pod security policies |
| GitHub Actions | CI/CD pipeline automation |
| Terraform | AWS infrastructure provisioned as code — reproducible, version-controlled |

### Security pipeline
| Tool | Stage | Why at this stage |
|---|---|---|
| SAST scanner | Pull request | Catches insecure code patterns before merge — cheapest point to fix |
| Trivy | Post-build | Scans container images for known CVEs before any deployment |
| DAST | Staging environment | Needs a running application to probe — run against staging, never production |
| Secrets scanner | Pre-commit + CI | Prevents credentials reaching the repository at all |

### Observability
| Tool | Role |
|---|---|
| Prometheus | Metrics collection — CPU, memory, request latency, error rates |
| Grafana | Dashboards and alerting thresholds |
| Centralized logging | Aggregated logs across all services for incident investigation |

---

## Security design decisions

### Why SAST at PR stage, not just pre-deploy?
Fixing a security issue at code review costs a fraction of fixing it post-deployment. Running SAST on every pull request means insecure patterns never reach the main branch.

### Why Trivy for container scanning over manual review?
Container images pull in hundreds of transitive dependencies. Trivy automates CVE detection across the full image layer stack — something no manual review can do at speed.

### Why DAST on staging and not production?
DAST actively probes a running application for vulnerabilities. Running it on production risks real user impact and triggers alerts. Staging gives a realistic target without the risk.

### Why Kubernetes secrets with a secrets manager over environment variables?
Environment variables leak into logs, crash reports, and process listings. A secrets manager gives rotation, audit trails, and least-privilege access. Env vars have none of these.

### Why RBAC at the Kubernetes level?
RBAC limits what each pod, service account, and user can do inside the cluster. Without it, a compromised container can access secrets and resources across the entire cluster.

---

## CI/CD pipeline

```
Code push / PR opened
        │
        ▼
  SAST scan + secrets scan
        │
        ▼ (pass)
  Automated tests
        │
        ▼ (pass)
  Docker image build
        │
        ▼
  Trivy container scan
        │
        ▼ (pass)
  Deploy to staging
        │
        ▼
  DAST scan on staging
        │
        ▼ (pass)
  Approval gate → deploy to production
        │
        ▼
  Rollback triggered automatically on failure
```

**Why an approval gate before production?**
Automated checks catch known issues. An approval gate ensures a human reviews the deployment intent, particularly for infrastructure changes where automation can be wrong in ways tools don't catch.

---

## Trade-offs made

| Decision | What was gained | What was traded |
|---|---|---|
| Kubernetes over serverless | Full control over pod security, network policies, RBAC | More operational complexity to manage |
| SAST at PR stage | Earlier detection, lower fix cost | Slightly slower PR review cycle |
| Terraform IaC over console provisioning | Reproducible, auditable infrastructure | Higher initial setup time |
| Microservices over monolith | Independent deployment and blast-radius containment | More inter-service complexity |
| PostgreSQL over NoSQL | Strong consistency for auth/user data | Less flexible for unstructured data |

---

## What is implemented

- [x] API gateway with authentication and rate limiting
- [x] Microservices architecture with independent deployments
- [x] Docker containerisation with image hardening
- [x] Kubernetes orchestration with RBAC and pod security
- [x] Terraform IaC for AWS infrastructure provisioning
- [x] CI/CD pipeline with GitHub Actions
- [x] SAST integration at pull request stage
- [x] Trivy container image scanning
- [x] Secrets management system
- [x] DAST scanning against staging environment
- [x] Prometheus metrics collection
- [x] Grafana dashboards
- [ ] OpenTelemetry distributed tracing — in progress
- [ ] Kubernetes network policies for pod-to-pod traffic control
- [ ] Supply chain security (SBOM generation, Sigstore signing)

---

## What I would add next and why

**Kubernetes network policies**
Currently RBAC controls who can do what, but pod-to-pod traffic is unrestricted within the cluster. Network policies would enforce that Service A can only talk to the database, not to Service B — least-privilege at the network level.

**SBOM generation**
A Software Bill of Materials gives a full inventory of every dependency in the build. With supply chain attacks increasing, knowing exactly what is in a container image matters as much as scanning it.

**OpenTelemetry tracing**
Prometheus tells you a request is slow. Distributed tracing tells you which service in the chain is causing it. For a microservices system this is the difference between guessing and knowing.

---

## Local setup

```bash
git clone https://github.com/Rahulkumar240/nexus-platform
cd nexus-platform
docker-compose up --build
```

Kubernetes deployment:
```bash
kubectl apply -f k8s/
```

Terraform provisioning:
```bash
cd terraform/
terraform init
terraform plan
terraform apply
```

---

## Author

**Rahul Kumar** — DevSecOps engineer-in-training, BTech CSE @ IKGPTU  
[LinkedIn](https://linkedin.com/in/rahulkumar297) · [GitHub](https://github.com/Rahulkumar240)

---

## License

MIT License
