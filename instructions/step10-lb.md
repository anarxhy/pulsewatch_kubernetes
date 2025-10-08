# Step 10: Configuring an LB

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml

# The service should be in the `metallb-system` namespace

kubectl get all -n metallb-system

Adding a metallb pool to match the docker network ( accessible via host)

```

curl -X GET localhost
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>web</title>
    <script type="module" crossorigin src="/assets/index-7X1zgiqF.js"></script>
    <link rel="stylesheet" crossorigin href="/assets/index-BdOndhxL.css">
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>

```

For a local kind cluster, we can use a ClusterIP service and ingress-nginx.

```