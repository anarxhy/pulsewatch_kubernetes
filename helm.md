# PulseWatch Deployment Guide

## Prerequisites
- **Kubernetes cluster** (v1.24+)
- **kubectl** configured with cluster access
- **Helm** (v3.8+)
- **Docker** (for building images)


## Deployment Steps

### 1. Create Namespace
```bash
kubectl create namespace pulsewatch
```

### 2. Deploy Database Layer

#### PostgreSQL
```bash
helm upgrade --install postgres ./postgres -n pulsewatch
```

**Verification:**
```bash
kubectl get pods -n pulsewatch -l app=postgres
kubectl exec -it postgres-0 -n pulsewatch -- psql -U postgres -c "\l"
```

#### Redis
```bash
helm upgrade --install redis ./redis -n pulsewatch
```

**Verification:**
```bash
kubectl get pods -n pulsewatch -l app=redis
kubectl exec -it deployment/redis -n pulsewatch -- redis-cli ping
```

### 3. Run Database Migrations

```bash
helm upgrade --install flyway ./flyway -n pulsewatch
```

**Verification:**
```bash
# Check job status
kubectl get jobs -n pulsewatch -w

# Check migration logs
kubectl logs -n pulsewatch -l job-name=flyway-migrate -c flyway

# Verify tables were created
kubectl exec -it postgres-0 -n pulsewatch -- psql -U postgres -d pulsewatch -c "\dt"
kubectl exec -it postgres-0 -n pulsewatch -- psql -U postgres -d pulsewatch -c "SELECT * FROM flyway_schema_history;"
```

### 4. Deploy Application Services

#### API Service
```bash
helm upgrade --install api ./api -n pulsewatch
```

**Verification:**
```bash
kubectl get pods -n pulsewatch -l app=api
kubectl logs -n pulsewatch deployment/api -f
```

#### Worker Service
```bash
helm upgrade --install worker ./worker -n pulsewatch
```

**Verification:**
```bash
kubectl get pods -n pulsewatch -l app=worker
kubectl logs -n pulsewatch deployment/worker -f
```

#### Web Service (Nginx)
```bash
helm upgrade --install web ./web -n pulsewatch
```

**Verification:**
```bash
kubectl get pods -n pulsewatch -l app=web
kubectl get services -n pulsewatch web
kubectl logs -n pulsewatch deployment/web
```

### 5. Deploy Ingress Controller (Optional - for external access)

```bash
# For Kind clusters or local development
kubectl apply -n ingress-nginx -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# Wait for ingress controller to be ready
kubectl -n ingress-nginx rollout status deploy/ingress-nginx-controller
```

### 7. Update Hosts File (Local Development)

Add to `/etc/hosts` (Linux/Mac) or `C:\Windows\System32\drivers\etc\hosts` (Windows):
```
127.0.0.1 pulsewatch.local
```

### 8. Access the Application

```bash


# Access via browser
open http://pulsewatch.local

```

## Complete Verification

```bash
# Check all resources
kubectl get all -n pulsewatch

# Check network policies
kubectl get networkpolicies -n pulsewatch

# Check persistent volumes
kubectl get pvc -n pulsewatch
```

## Maintenance Commands

### Restart Services
```bash
kubectl rollout restart deployment/api -n pulsewatch
kubectl rollout restart deployment/web -n pulsewatch  
kubectl rollout restart deployment/worker -n pulsewatch
```

### View Real-time Logs
```bash
kubectl logs -n pulsewatch deployment/api -f
kubectl logs -n pulsewatch deployment/worker -f
kubectl logs -n pulsewatch deployment/web -f
```

### Database Backup
```bash
kubectl exec -it postgres-0 -n pulsewatch -- pg_dump -U postgres pulsewatch > backup_$(date +%Y%m%d).sql
```

## Cleanup

```bash
# Uninstall all Helm releases
helm uninstall api web worker redis postgres flyway -n pulsewatch

# Delete namespace (this will remove everything)
kubectl delete namespace pulsewatch

# Clean up ingress controller (if installed)
kubectl delete -n ingress-nginx -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

---

