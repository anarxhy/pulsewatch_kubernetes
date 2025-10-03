# Step 01: Postgres (secure StatefulSet)

## StatefulSet + PVC
I use a StatefulSet to get a stable identity and attach persistent storage with a volumeClaimTemplates (2 GiB). This relies on the cluster’s default StorageClass.

## Service name = postgres
serviceName: postgres pairs with a Service (ClusterIP) so other pods can reach postgres:5432.


## Env from Secret
Credentials come from a Secret named postgres-auth (created separately). This keeps passwords out of the manifest.

## PGDATA subdirectory
I set PGDATA=/var/lib/postgresql/data/pgdata. Using a subfolder avoids some permission quirks on fresh volumes.

## Probes

- readiness: pg_isready against the target DB → only accept traffic when ready.

- liveness: same check, with a longer initial delay to avoid flapping during start.

## Security context (light)

- allowPrivilegeEscalation: false to satisfy restricted PodSecurity guidance.

- runAsUser: 999 / runAsGroup: 999 so the container runs non-root (works smoothly with postgres image on dev clusters).


## Apply Postgres Configuration

```
kubectl apply -f deploy/k8s/dev/10-postgres-secret.yaml
kubectl apply -f deploy/k8s/dev/11-postgres-statefulset.yaml
kubectl apply -f deploy/k8s/dev/12-postgres-service.yaml
kubectl apply -f deploy/k8s/dev/13-postgres-networkpolicy.yaml

kubectl -n pulsewatch rollout status statefulset/postgres
kubectl -n pulsewatch get pods -l app=postgres
```


## Verify Postgres²
```
kubectl -n pulsewatch exec -it statefulset/postgres -- \
  psql -U postgres -d pulsewatch -c "select version();"
```