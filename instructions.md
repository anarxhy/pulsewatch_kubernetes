# To install kind on Linux

curl -Lo kind https://kind.sigs.k8s.io/dl/v0.24.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/kind
kind version

# To check nodes 
kubectl get nodes 

# To delete a cluster
kind delete cluster --name pulsewatch

# To use kubectl with the cluster
kubectl cluster-info --context kind-pulsewatch

# Create cluster

kind create cluster --name pulsewatch --config deploy/k8s/cluster/kind.yaml




# To load local images into the cluster
kind load docker-image <image-name> --name pulsewatch

kind load docker-image pulsewatch-api:prod   --name pulsewatch
kind load docker-image pulsewatch-web:prod   --name pulsewatch
kind load docker-image pulsewatch-worker:prod --name pulsewatch

kind load docker-image pulsewatch-migrations:dev   --name pulsewatch
# deploy ingress-nginx

kubectl apply -n ingress-nginx -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

kubectl -n ingress-nginx rollout status deploy/ingress-nginx-controller

# Deploy k8s ressources: 

kubectl apply -f deploy/k8s/dev/


# Host: 

Add the following line to /etc/hosts or C:\Windows\System32\drivers\etc\hosts

127.0.0.1   pulsewatch.local
# Access the app