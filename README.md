# pulsewatch_kubernetes
PulseWatch on Kubernetes, Architecture &amp; Migration Plan

# Goals

- Reproducible deploys: versioned Helm releases, deterministic DB migrations (Flyway).

- Safe rollouts: probes, PDBs, HPAs, canaries when needed.

- Security first: non-root, read-only FS, least privilege, NetworkPolicies, secret management.

- Observability: metrics, logs, traces with sane defaults.

- Scalable: horizontal autoscaling, stateless services; managed data stores in prod.


# Target architecture (baseline)


- Namespaces: one per env (dev, staging, prod) + platform for shared addons.

- Ingress: NGINX or AWS ALB (cloud env); TLS via cert-manager/ACM.

- Web: static SPA (Nginx container) behind Ingress (or CloudFront in cloud).

- API: Express deployment, ClusterIP service.

- Worker: Deployment (or CronJob if needed), no external exposure.

- Migrations: Flyway Job per release; blocks app rollout until success.

- Redis: Deployment/Service in dev; managed (Elasticache/Memorystore) in prod.

- Postgres: StatefulSet in dev; managed (RDS/CloudSQL) in prod.

- Secrets: External Secrets Operator (AWS Secrets Manager/SSM) in cloud; Secret in dev.

- Observability: kube-prometheus-stack + Loki (or CloudWatch) + OpenTelemetry Collector.

- Policies: default-deny NetworkPolicies; Pod Security baseline (warn restricted).



| Area           | Choice                                                   | Why                                                          |
| -------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| Packaging      | **Helm** per service (+ optional umbrella)               | Widely used, values per env, simple CI.                      |
| GitOps (later) | Argo CD or Flux                                          | Declarative, PR-driven ops. Start with Helm CLI.             |
| DB migrations  | **Flyway Job** baked with SQL                            | Checksums, audited, app-independent.                         |
| Secrets        | **External Secrets** in cloud; raw `Secret` in dev       | No plaintext in repo; easy local dev.                        |
| Ingress        | NGINX (dev), ALB (AWS)                                   | Simplicity locally; native in cloud.                         |
| Storage        | PVCs in dev; **Managed** in prod                         | Reliability + ops burden reduction.                          |
| Autoscaling    | HPA (CPU+requests), later VPA/Karpenter in cloud         | Cost + resilience.                                           |
| Security       | Non-root, readOnlyRootFS, drop caps, NetworkPolicy, PDBs | Baseline hardening.                                          |
| Observability | kube-prometheus-stack + Loki + OpenTelemetry Collector    | Standard, flexible, extensible.                               |
| CI/CD          | GitHub Actions (later GitOps)                            | Familiar, free for OSS, extensible.                          |


# Container expectations (all services)

- Non-root user, readOnlyRootFilesystem: true, drop all capabilities.

- Health: readiness and liveness probes:

- API: GET /health on 3001

- Web: GET / on 80 (static)

- Worker: exec health or lightweight HTTP /health

## Resources (starters):

- API req 100m/128Mi, lim 500m/512Mi; Worker 50m/96Mi, 300m/256Mi; Web 50m/64Mi, 300m/256Mi.
- Config: env via values.yaml → ConfigMap/Secret; no hardcoded endpoints.
- Images: multi-stage, minimal base (distroless/alpine), no root, pinned versions.


## Helm layout
```
pulsewatch-k8s/
├─ charts/
│  ├─ api/
│  ├─ worker/
│  ├─ web/
│  ├─ redis/          # optional: use bitnami for speed
│  └─ migrations/     # Flyway Job chart (image + args)
├─ envs/
│  ├─ dev/values.yaml
│  ├─ staging/values.yaml
│  └─ prod/values.yaml
├─ platform/          # ingress, cert-manager, monitoring (phase 2)
└─ docs/
   ├─ ARCHITECTURE.md
   └─ DECISIONS.md
```

# Per-chart must-haves

- Deployment/Service (or Job), ServiceAccount, NetworkPolicy, PDB, HPA, values.yaml.

- SecurityContext + probes + resource requests/limits.

- Ingress only for Web/API (dev: optional).

# Config & secrets (env-specific)

- envs/dev/values.yaml: inline DATABASE_URL, REDIS_URL, JWT_SECRET (or mount from Secret).

- envs/prod/values.yaml: reference External Secrets (e.g., external-secrets.io CRDs) that pull from AWS SM/SSM.
