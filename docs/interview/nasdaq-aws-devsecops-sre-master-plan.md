# Nasdaq R0026713 — AWS Platform Engineering, DevSecOps & SRE Master Plan

**Context:** Technical panel, Thursday October 22, 2026, 3:30–4:30 PM ET. Panel includes software engineers; expect code-to-production reasoning, architectural tradeoffs, debugging and behavioral questions. This is an independent fictional capital-markets platform, NOT Nasdaq internal architecture. Synthetic market data only.

## Definition of done
Three independent services (market-data-api, portfolio-api, risk-api) deployed to Amazon EKS with Terraform-managed AWS foundations, Docker/ECR, Jenkins CI, Helm + Argo CD GitOps, security quality gates, measurable reliability objectives, alerts, tested rollback and documented failure recovery. Every milestone includes implementation, tests/evidence, troubleshooting, tradeoffs, and an interview explanation. Nothing planned should be represented as deployed.

## Company architecture and ownership
Developers -> GitHub PR -> Jenkins CI (unit/integration tests + SAST + SCA + secrets + IaC scan) -> Docker build -> Trivy image scan + SBOM/signing -> Amazon ECR -> GitOps repo/chart version change -> Argo CD -> Helm -> EKS namespaces/services -> OpenTelemetry/Prometheus/Grafana/CloudWatch -> alerts and incident response.

Platform team owns reusable Terraform, secure cluster, developer templates, CI standards, GitOps and observability. Application teams own service code, tests, instrumentation, SLOs and operational runbooks. Security owns policies and exceptions; SRE partners on reliability, incidents and capacity. No production change without review.

## 1. Terraform deeply (AWS)
HCL syntax, provider/version pinning, locals, expressions, variable validation, outputs, modules, dependency graph, data sources, count/for_each, lifecycle, plan/apply/destroy, import/moved blocks, drift detection, state inspection and troubleshooting. Build reusable modules for VPC, subnets, route tables, IGW, NAT, VPC endpoints, SGs, IAM roles/OIDC, ECR, EKS managed node groups, KMS, CloudWatch and alarms. Distinct dev/stage/prod root modules and remote S3 state with native S3 locking where supported; encryption, versioning, least-privilege IAM and state recovery. Explain difference between provider dependency lock file and state lock. Plan/apply only via approved CI; avoid manual console drift.

Practice questions: Why is Terraform plan proposing replacement? What happens if apply partially fails? How do you resolve locked state safely? How do you import pre-existing AWS resources? How do you detect and reconcile drift? What is your module/versioning strategy?

## 2. Kubernetes, pods, workloads and debugging
Control plane: API server, etcd, scheduler, controllers. Worker node: kubelet, container runtime, kube-proxy/CNI, CoreDNS. Pod lifecycle Pending -> Running -> Succeeded/Failed; init containers, sidecars, restart policy; CrashLoopBackOff vs ImagePullBackOff vs OOMKilled vs Pending; readiness, liveness, startup probes; requests/limits; QoS, evictions; node affinity/taints/tolerations; namespaces, service accounts, RBAC, network policies, Secrets via AWS Secrets Manager integration; deployments, ReplicaSets, rolling updates, services, ingress/ALB controller, DNS, HPA, cluster autoscaling, PDB, PVC/storage classes, EBS CSI, EKS IAM Pod Identity/IRSA and ECR pull permissions.

Debugging path: kubectl get pods -A -o wide -> describe pod -> logs --previous -> get events -> inspect probes, env, config, IAM, image and registry -> check service endpoints/DNS/network policy/ingress -> node capacity and autoscaler -> roll back safely. Practice a 5xx error spike after a rollout and distinguish application failure, service routing, probe failures and capacity saturation.

## 3. Software engineering and microservices
Implement three small APIs with health/readiness endpoints, request IDs, structured logs, unit and integration tests, versioned contracts, graceful shutdown, timeouts/retries with backoff, circuit breaker awareness, idempotency, connection pools and dependency failure behavior. Explain Docker multi-stage builds, non-root user, pinned base images, small attack surface and immutable tags/digests. Design failure-resistant release strategy with Helm values per environment, Argo CD sync policy, progressive rollout/canary options and rollbacks.

## 4. Jenkins CI/CD and DevSecOps
GitHub webhook -> Jenkins multibranch PR pipeline -> lint/unit tests -> secret scan (Gitleaks) -> SAST (Semgrep/SonarQube) -> SCA (Dependency-Check or equivalent) -> Terraform fmt/validate/Checkov/tfsec -> container build -> Trivy scan -> SBOM (Syft) -> provenance/signature (Cosign where practical) -> push to ECR -> update GitOps manifest by PR -> Argo CD deploy -> smoke/integration/DAST tests in isolated environment -> approvals for promotion. Distinguish CI artifact creation from GitOps deployment. Set severity thresholds, exceptions with expiration, secure credentials/OIDC, least privilege and auditable change records. Practice failure gates, pipeline rollback, compromised dependency, exposed secret and flaky tests.

## 5. Site Reliability Engineering: SLIs, SLOs, SLAs and error budgets
- SLI = measured indicator such as successful HTTP request ratio, p95 latency, saturation or availability.
- SLO = internal reliability target measured over a window; example 99.9% good requests over rolling 30 days.
- SLA = external contractual promise, often with consequences; do not equate SLA with SLO.
- Error budget = allowed failure proportion; at 99.9% success SLO, budget is 0.1% bad requests (1,000 per 1,000,000). For an availability-based 30-day month, 0.1% is 43.2 minutes, but request-based budgets depend on request counts, not minutes.
- Example latency SLI: proportion of eligible requests under 300 ms; target 99% over 28 days.
- Example availability SLI: (eligible requests excluding explicitly defined client errors - server failures) / eligible requests; specify treatment of 4xx, timeouts, and health-check traffic.
- Define burn-rate alerting across short and long windows, and response actions when error budget is exhausted (pause risky releases, prioritize reliability).
- Understand error rate = failed requests / total eligible requests; HTTP 5xx spikes, latency percentiles, RED (Rate, Errors, Duration), USE (Utilization, Saturation, Errors), golden signals (latency, traffic, errors, saturation).
- Observability: OpenTelemetry traces, Prometheus metrics, Grafana dashboards, CloudWatch logs/alarms, Alertmanager; correlation IDs across microservices.
- On-call: incident severity, detection, triage, mitigation, rollback, communication, postmortem and follow-up actions. Explain MTTR, MTTD, RTO, RPO, redundancy across availability zones, backups and disaster recovery.
- Capacity: CPU/memory requests/limits, HPA, cluster autoscaler, load testing, bottlenecks, cost/reliability tradeoffs.

## 6. Platform engineering and developer experience
Self-service developer golden paths: Backstage software templates or simple CLI scaffolder; standardized repo, CI pipeline, Helm chart, service account, metrics, logs, security defaults, deployment environment and runbook. Offer paved roads without hiding necessary control. Track developer onboarding time, deployment lead time, change failure rate, deployment frequency, recovery time, adoption and security exceptions. Discuss build-vs-buy, governance, tenant isolation, multi-team boundaries and documentation.

## 7. AWS security and operations
AWS Organizations/account separation as feasible, IAM least privilege, federation/OIDC, EKS workload identity, KMS, Secrets Manager, CloudTrail, GuardDuty (cost permitting), ECR scanning, encrypted S3, network segmentation, restricted endpoints, WAF where appropriate, cost budgets/alerts and cleanup. Avoid using real market, customer or employer data.

## 8. Behavioral / software-engineer panel preparation (STAR)
Prepare specific and truthful examples with Situation, Task, Action, Result and tradeoffs:
1. Terraform module design or state/drift incident and how you resolved it.
2. Kubernetes outage: CrashLoopBackOff, bad probe, image pull or OOM and the diagnostic sequence.
3. CI/CD improvement: measurable deployment speed, failure rate or security coverage (use only documented real metrics).
4. Disagreement with developers about security gates or rollout risk.
5. Incident ownership under pressure: communicate, mitigate, follow up.
6. Migrating legacy workloads while minimizing downtime and risk.
7. Mentoring engineers, code reviews, standards and developer self-service.
8. Handling ambiguous requirements, priorities and stakeholder expectations.
9. A failed change, accountability and lessons learned.
10. Why Nasdaq / why platform engineering: secure, reliable shared infrastructure that improves developer velocity.

Answer as a Lead Engineer: explain business need, system design, what YOU did, why, alternatives, verification, and impact. Don't invent numbers or claim unimplemented project phases as professional experience.

## Tonight: priority order for 60-minute technical panel tomorrow
P0: Validate repo, Terraform state/modules, AWS identity and cost guardrails.
P0: Deploy one small container locally and prepare EKS/pod troubleshooting examples.
P0: Walk through Jenkins -> ECR -> Helm -> Argo CD architecture and security gates.
P0: Calculate an SLO/error budget and troubleshoot 5xx error rate spike.
P0: Practice at least five behavioral STAR stories aloud.
P1: Full EKS cluster, Argo CD and Prometheus/Grafana if time and budget permit.
P2: Backstage and advanced canary/chaos/DR once core platform works.

## Verification checklist
- [ ] AWS identity, region and budget verified
- [ ] Terraform validate and plan artifacts saved
- [ ] EKS cluster and pod health verified
- [ ] CI pipeline executed with scanning results
- [ ] Helm release and Argo CD sync evidenced
- [ ] Synthetic traffic generated, SLI dashboard and alert tested
- [ ] Failure drill, rollback and postmortem documented
- [ ] Interview answers distinguish completed from planned work
