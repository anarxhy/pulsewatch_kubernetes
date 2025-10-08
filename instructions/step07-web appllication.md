## step 06: Setting up web appllication deployment on kubernetes

Fistt we need to build the web image. Check out my project pulsewatch (web/Dockerfile)

docker build -t pulsewatch-web:dev apps/web



3) Creating deployment for the web app
4) Creating service for the web app

Allow to Postgres 5432, Redis 6379, API 3001, and public internet (80/443) so users can reach it.


# apply and verify

kubectl apply -f deploy/k8s/dev/60-web-deployment.yaml
kubectl apply -f deploy/k8s/dev/61-web-service.yaml
kubectl -n pulsewatch rollout status deploy/web




For Now, We will forward the port to access the dev web app (we will later use nginx to serve the web app on 80/443)
# Web (Vite)
kubectl -n pulsewatch port-forward svc/web 5173:5173
# API
kubectl -n pulsewatch port-forward svc/api 3001:3001


# When testing locally, we will find that the web app cannot reach the API due to NetworkPolicy restrictions.
# To quickly test, we can temporarily allow all traffic to the API (not recommended for production).

# The issue was in network policy for Wokrer, Redis 

in 43-api-networkpolicy.yaml, we allow ingress from web on port 3001
```
ingress:
    - from:
        - podSelector:
            matchLabels:
              app: web
      ports:
        - protocol: TCP
          port: 3001
```

Adding redis network policy to allow traffic from API and Worker

# API: allow ingress from web, and egress to Postgres + Redis
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-policy
  namespace: pulsewatch
spec:
  podSelector:
    matchLabels:
      app: redis
  policyTypes: ["Ingress"]
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api
      ports:
        - protocol: TCP
          port: 6379
 






