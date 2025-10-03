# Step 08: Exposing the application 

We wil use an LB ( MetalLB) 
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
# Configure an address pool in kind’s subnet if not done earlier

```

Patch the svc web to type LoadBalancer ( if not working, update the web service directly and recreate it)
```
kubectl -n pulsewatch patch svc web \
  --type merge -p '{"spec":{"type":"LoadBalancer","loadBalancerClass":"metallb"}}'
```
## Result 
```
NAME   TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
web    LoadBalancer   10.96.22.199   172.18.255.200   5173:30091/TCP   4s

```