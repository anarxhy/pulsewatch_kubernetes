# Improving app connectivity: 
Vite proxy to API 

to avoid running the following command every time:
```
kubectl -n pulsewatch port-forward svc/web 5173:5173
kubectl -n pulsewatch port-forward svc/api 3001:3001
```

Add the following to vite.config.js ( For DEV)
```
export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': 'http://localhost:3001',
    },
  },
})
```
This will proxy all /api requests to the API service.


For production, we will use NGINX ingress to route traffic to the web and API services.

# Step 09: Exposing the application with Ingress

The application is now exposing via port 80 ( web) and 3001 ( API) inside the cluster.
We need to update all the network policies to allow traffic to port 80 instead 
(update in code and redeploy)


After making the changes, The application should be accessible 

```
kubectl port-forward svc/web 8080:80 -n pulsewatch
http://localhost:8080
```

We can achieve the same thing using NodePort services, since we are using a local cluster, (ExternalPort should be also configured ( restart cluseter))

but Ingress is more flexible and powerful.