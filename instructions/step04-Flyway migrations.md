## Step 3 — Flyway migrations (K8s Job)

First we need to run DB migrations. We’ll use Flyway, a popular migration tool that works well with Postgres.

As a starting point, we'll recreate our migration docker image.

# build image from ops/migrations/Dockerfile (contains V1__, V2__, …)$

docker build -t pulsewatch-migrations:dev ops/migrations

kubectl apply -f deploy/k8s/dev/30-db-flyway-secret.yaml
kubectl apply -f deploy/k8s/dev/31-db-migrate.yaml
kubectl apply -f deploy/k8s/dev/32-allow-flyway-networkpolicy.yaml



## verify migrations ran

```
kubectl exec -n pulsewatch postgres-0 -- psql -U postgres -d pulsewatch -c "\dn"

kubectl exec -n pulsewatch postgres-0 -- psql -U postgres -d pulsewatch -c "
SELECT installed_rank, version, description, type, script, installed_by, installed_on, execution_time, success 
FROM flyway_schema_history 
ORDER BY installed_rank;"

```