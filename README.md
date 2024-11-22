# hub-consulcatalog

Reproducible env for Hub and consulCatalog

## Set up K8s env

```bash
k3d cluster create repro --port 80:80@loadbalancer --port 443:443@loadbalancer --port 8000:8000@loadbalancer --k3s-arg "--disable=traefik@server:0"
```

## Set up Traefik Hub

### Create Traefik Hub namespace

```bash
kubectl create ns traefik
```

### Set Traefik Hub token

```bash
kubectl create secret generic hub-license --from-literal=token="${HUB_TOKEN}" -n traefik
```

### update helm repo if already present

```bash
helm repo update
```

### Deploy traefik

```bash
helm upgrade --install traefik traefik/traefik --create-namespace --namespace traefik --values hub/hub-values.yaml
```

### Set Dashboard ingress

```bash
kubectl apply -f hub/dashboard.yaml
```

## Set up Consul

```bash
helm upgrade -i consul hashicorp/consul --create-namespace --namespace consul --values consul/consul-values.yaml
```

### Set Dashboard ingress

```bash
kubectl apply -f consul/dashboard.yaml
```

## Deploy test app

```bash
kubectl apply -f whoami
```

## Test Traefik error code

### K8s ingress with allowEmptyServices

```bash
curl http://whoami-k8sprov.docker.localhost -i
```

Result:

```txt
HTTP/1.1 503 Service Unavailable
Content-Type: text/plain; charset=utf-8
X-Content-Type-Options: nosniff
Date: Fri, 22 Nov 2024 10:24:24 GMT
Content-Length: 20

no available server
```

### consulCatalog route without allowEmptyServices

```bash
curl http://whoami-consprov.docker.localhost -i
```

Result:

```txt
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=utf-8
X-Content-Type-Options: nosniff
Date: Fri, 22 Nov 2024 10:26:11 GMT
Content-Length: 19

404 page not found
```

## Destroy env at the end

```bash
k3d cluster stop repro 
k3d cluster delete repro
```
