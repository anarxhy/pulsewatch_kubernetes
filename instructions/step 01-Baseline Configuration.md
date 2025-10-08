# Step 00: Baseline Configuration


Note: For kubernetes cluster, we wil use kind (Kubernetes IN Docker) for local development and testing.

1) Namespace with Pod Security labels

- New namespace pulsewatch for the demo
- Pod Security labels set to privileged 

2) Default requests/limits + quotas

3) Network Policies (zero trust by default)

We’ll add explicit allow policies per app later (Postgres, Redis, API, Worker, Web).
Note: some local clusters don’t enforce NetworkPolicy—still keep these for portability.

4) ServiceAccounts (principle of least privilege)

## Apply Check Configuration

kubectl apply -f deploy/k8s/dev/00-namespace.yaml
kubectl apply -f deploy/k8s/dev/01-quotas.yaml
kubectl apply -f deploy/k8s/dev/02-network-baseline.yaml
kubectl apply -f deploy/k8s/dev/03-serviceaccounts.yaml

## Verify Configuration

kubectl get ns pulsewatch -o jsonpath='{.metadata.labels}'
kubectl -n pulsewatch get limitrange,resourcequota
kubectl -n pulsewatch get networkpolicy
kubectl -n pulsewatch get sa

