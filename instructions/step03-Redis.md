# Redis — Architecture Choices & Decisions (PulseWatch)

This doc explains **why and how** we use Redis in PulseWatch, with clear dev vs. prod guidance.

---

## 1) What Redis is used for

- **Rate limiting** (IP/user): fast counters with TTL.
- **JWT blacklist** (logout): short-lived deny-list keys.
- **Cache** (read-through for `/users`, future dashboards).
- *(Optional later)*: background job queues / dedupe.

> No long-term data. Losing Redis in **dev** is acceptable; in **prod** we use managed, HA Redis.

---

## 2) Why Redis (and not a DB-only solution)

- **Low latency** (single-digit ms), atomic increments, expirations, and Lua when needed.
- **Simplicity**: built-in primitives fit rate limiters and token revocation perfectly.
- **Cost/perf**: offload hot paths from Postgres.

Alternatives considered:
- **Postgres only** – simpler stack but slower for counters, requires cleanup jobs.
- **Memcached** – lacks durable structures and fine-grained TTL/ops we need.
- **Redis Streams** – interesting for jobs, but we’ll keep queues optional for now.

---

## 3) Deployment choices

### Dev (kind / Docker Desktop / Compose)
- **Container**: `redis:7`
- **Persistence**: **disabled** (`--appendonly no`) — faster restarts; data can be lost.
- **Access**: in-cluster via `redis:6379`; from Compose via `redis://redis:6379`.

Kubernetes (dev) minimal (already in repo):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: redis, namespace: pulsewatch }
spec:
  replicas: 1
  selector: { matchLabels: { app: redis } }
  template:
    metadata: { labels: { app: redis } }
    spec:
      containers:
        - name: redis
          image: redis:7
          args: ["--appendonly", "no"]
          ports: [{ name: redis, containerPort: 6379 }]
          readinessProbe: { exec: { command: ["redis-cli","ping"] }, initialDelaySeconds: 3 }
          livenessProbe:  { exec: { command: ["redis-cli","ping"] }, initialDelaySeconds: 10 }
---
apiVersion: v1
kind: Service
metadata: { name: redis, namespace: pulsewatch }
spec:
  selector: { app: redis }
  ports: [{ name: redis, port: 6379, targetPort: 6379 }]
