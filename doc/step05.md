## Step 05: Worker on kubernetes

First we start building the worker image. check out my project poulsewatch ( worker/Dockerfile)

docker build -t pulsewatch-worker:dev apps/worker

1) Creating secrets for the worker app
2) Creating configmap for the worker app
3) Creating deployment for the worker app
4) Creating network policy to allow traffic to the worker app

Allow to Postgres 5432, Redis 6379, and public internet (80/443) so probes can reach targets.


# apply and verify

```
kubectl apply -f deploy/k8s/dev/50-worker-secrets.yaml
kubectl apply -f deploy/k8s/dev/51-worker-config.yaml
kubectl apply -f deploy/k8s/dev/52-worker-deployment.yaml
kubectl apply -f deploy/k8s/dev/53-worker-networkpolicy.yaml

kubectl -n pulsewatch rollout status deploy/worker
kubectl -n pulsewatch logs deploy/worker --tail=100 -f
kubectl -n pulsewatch get pods -l app=worker
``` 


