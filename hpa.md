# Deploy two HPAs for api and web

kubectl apply deploy/k8s/hpa/hpa.yaml

# Check HPAs

kubectl get hpa -A

```
amine@DESKTOP-STB6FHU:/mnt/c/Users/amine/Desktop/projects/pulsewatch_kubernetes$ kubectl get hpa -A 
NAMESPACE    NAME     REFERENCE           TARGETS              MINPODS   MAXPODS   REPLICAS   AGE
pulsewatch   api      Deployment/api      cpu: <unknown>/70%   2         6         2          49s
pulsewatch   web      Deployment/web      cpu: <unknown>/70%   2         6         2          49s
pulsewatch   worker   Deployment/worker   cpu: <unknown>/70%   2         6         2          49s
```


# Enable metrics-server

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl rollout restart deployment/metrics-server -n kube-system --context=pulsewatch

# Check metrics-server logs

kubectl get pods -n kube-system  | grep metric

kubectl get hpa -n pulsewatch
NAME     REFERENCE           TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
api      Deployment/api      cpu: 3%/70%   2         6         2          5m52s
web      Deployment/web      cpu: 1%/70%   2         6         2          5m52s
worker   Deployment/worker   cpu: 2%/70%   2         6         2          5m52s

# Testing HPA using load script

kubectl get hpa -n pulsewatch
NAME     REFERENCE           TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
api      Deployment/api      cpu: 3%/3%    2         6         3          15m
web      Deployment/web      cpu: 1%/70%   2         6         2          15m

