## Step 4 — API app in kubernetes

We will now deploy the API app, which serves the backend.

# build image from api/Dockerfile (contains Express app)

docker build -t pulsewatch-api:dev api


1) Creating secrets for the API app
2) Creating deployment for the API app
2) Creating service for the API app
3) Creating network policy to allow traffic to the API app



```
# (for kind) make sure your image is loaded
kind load docker-image pulsewatch-api:dev --name kind

kubectl apply -f deploy/k8s/dev/40-api-secrets.yaml
kubectl apply -f deploy/k8s/dev/41-api-deployment.yaml
kubectl apply -f deploy/k8s/dev/42-api-service.yaml
kubectl apply -f deploy/k8s/dev/43-api-networkpolicy.yaml

kubectl -n pulsewatch rollout status deploy/api

# quick port-forward to hit it from your laptop
kubectl -n pulsewatch port-forward svc/api 3001:3001
curl -s http://localhost:3001/health


{"status":"ok","timestamp":"2025-10-01T16:21:35.485Z"}

```
