# ATLAS Capital Markets — Enterprise Platform Engineering

> **Independent learning project.** A fictional capital-markets technology organization inspired by the public job requirements for Nasdaq Lead Engineer, Platform Engineering & Reliability (R0026713). This is **not** Nasdaq's architecture, code, data, endorsement, or a representation of internal systems. All workloads and market data are synthetic.

## Business objective
Build a reusable internal developer platform supporting independent product teams delivering three services: `market-data-api`, `portfolio-api`, and `risk-api`. Teams request approved environments, ship auditable releases and maintain service-level objectives without separately reinventing infrastructure or security.

## Architecture journey (AWS-only for this project)
Product team -> Git feature PR -> code, secrets, dependency and IaC checks -> required reviews -> shared Terraform modules -> isolated development/staging/production remote states -> AWS account foundation, VPC (AZs, public and private subnets, NAT, endpoints, routing, security groups), IAM/OIDC, ECR, EKS with managed node groups -> CI builds/tests/scans/SBOM/signing -> Helm chart + Argo CD GitOps -> mock product services -> CloudWatch, OpenTelemetry, Prometheus, Grafana, alerting, SLOs, runbooks and recovery tests.

AWS is the sole deployment target for this project. Azure is out of scope. The implementation emphasizes Terraform, Kubernetes pods and diagnostics, secure CI/CD, error rates, SLIs/SLOs/SLAs, error budgets, and site reliability engineering.

**Detailed Nasdaq technical panel and build plan:** [AWS Platform Engineering, DevSecOps & SRE Master Plan](docs/interview/nasdaq-aws-devsecops-sre-master-plan.md).

## Repository roadmap
```text
.github/workflows/              PR quality/security checks, Terraform plan, artifact-build gates
.github/CODEOWNERS              Required owners and code-review routing
docs/architecture/              Architecture decisions, networks, identity, threat model
docs/interview/                 Interview question + explanation per milestone
docs/runbooks/                  Incidents, drift remediation, SLOs, rollback, DR
infra/bootstrap/                Create state buckets / initial access (separate bootstrap)
infra/modules/aws/              Reusable vpc, eks, ecr, iam, observability modules
infra/environments/dev/          Isolated Terraform root config and state
infra/environments/stage/        Isolated Terraform root config and state
infra/environments/prod/         Isolated Terraform root config and state
apps/market-data-api/           Synthetic market-pricing data; not live feeds
apps/portfolio-api/             Synthetic holdings / positions; no real accounts
apps/risk-api/                  Synthetic risk scoring; no financial advice or trading
platform/backstage/             IDP scaffolder templates and golden paths
platform/helm/                  Versioned Kubernetes deployment charts
platform/argocd/                GitOps Application definitions
security/                       Policy-as-code, scanning, evidence, exception templates
observability/                  OpenTelemetry, Prometheus, Grafana and SLO config
tests/                          Unit, integration, policy and failure-game-day tests
```

## Git workflow and branch strategy
- `main`: protected release-grade trunk; reviewed, reproducible changes only.
- `develop`: integration branch for progressive platform implementation.
- `feature/<ticket>-<topic>`: short-lived, single-change branches, e.g. `feature/PLAT-001-terraform-state`, targeting `develop`.
- `release/<version>`: created when promoting an integrated release; merged/tagged on `main`.
- `hotfix/<ticket>`: created from `main` for urgent production fixes; back-merged into `develop`.
- Do not create permanent branches per engineer or environment. Environment promotion uses reviewed configuration and GitOps, not manually changing Kubernetes.

**Repository policy to configure in GitHub:** require PR (no direct pushes), at least one review (2 recommended for production), CODEOWNERS for critical paths, checks for format/lint/tests/secret scan/SAST/SCA/Terraform validate/Checkov, up-to-date branches, conversation resolution, and restricted deploy environments. Adding workflow files alone does not enforce branch protection; protection/rulesets must be configured separately by a repository administrator.

## Terraform and state policy
- Code is the *desired configuration*, remote state maps resources to Terraform-managed objects, and cloud reality is what actually exists. The plan compares these during refresh.
- Each account/environment/root has a distinct state backend/key and least-privilege deploy identity.
- Protect state with access controls, encryption, versioning, audit logging and locking; never commit state, secrets or tfvars containing real secrets.
- `.terraform.lock.hcl` pins provider dependency versions. It is **not** the lock for concurrent state writes. Backend state locking protects against simultaneous writers. A state lock conflict is **not itself** infrastructure drift; drift occurs when real infrastructure differs from Terraform's expected configuration/state, e.g. console changes.
- Adopt a documented process for out-of-band changes: inspect plan, identify owner, decide revert/import/update configuration, review and apply via approved pipeline.

## Milestone sequence and interview practice
0. Git setup, SSH/HTTPS, trunk/develop/feature workflow, PR reviews, pre-commit security.
1. Terraform foundations: HCL, modules, variables, locals, outputs, providers, expressions, graph, plan/apply/destroy, backends, state, locks, drift, import/moved blocks, workspaces, lifecycle, governance.
2. AWS landing zone, VPC CIDRs/subnets, route tables, NAT, IGW, private endpoints, SGs/NACLs, KMS, IAM OIDC, ECR and EKS clusters.
3. Kubernetes control plane, scheduler, API server, etcd, CNI, CoreDNS, kube-proxy, nodes, kubelet, container runtime, deployments, services, ingress, scaling, PDB, storage, RBAC.
4. Three synthetic APIs, service boundaries, Dockerfiles, tests, Helm charts and Argo CD GitOps promotion.
5. DevSecOps: secret/SAST/SCA/container/IaC/DAST scans, SBOM, signing/provenance, approvals, vulnerability remediation.
6. Developer platform golden paths/Backstage, automation standards, policy-as-code and self-service.
7. CloudWatch, OpenTelemetry, Prometheus/Grafana, SLIs/SLOs/SLAs, error budgets, error-rate alerting, incident response, DR.
8. Deep Kubernetes pod debugging, reliability game days, platform engineering golden paths and behavioral STAR scenarios.
9. Failure injection, architecture reviews, team leadership and 60-minute mock technical panel.

At **every milestone**: one deployable change, one proof (test/log/plan/screenshot), one troubleshooting scenario, one `Why did we choose this?` trade-off, and interview questions answered conversationally.

## Status / safety
**Started:** architecture charter; Git structure to be built iteratively.
**Not yet verified:** branch protection, Terraform plans, AWS provisioning, security scanner gates, Kubernetes deployments and observability. Never represent an untested example as live production.

Use synthetic data only, cloud spending guardrails, named sandbox accounts, least-privilege credentials, and no employer-internal infrastructure or PHI.
