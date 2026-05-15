<div align="center">

# Proxy

**A Flux GitOps repository for the edge proxy Kubernetes cluster.**

[GitOps](#gitops) · [Routing](#routing) · [Network Bootstrap](#network-bootstrap) · [Bootstrap](#bootstrap)

</div>

---

This repository manages the desired state of the `homelab-proxy` Kubernetes cluster.

It runs Flux on K3s and publishes a small set of proxy-facing Kubernetes Service and EndpointSlice resources. The cluster acts as an edge/proxy plane that forwards selected public ports to backend services reachable over the private network.

## GitOps

- `clusters` is the Flux bootstrap path.
- `clusters/proxy.yaml` syncs the workload manifests from `./proxy`.
- `proxy/kustomization.yaml` lists the namespace, service, and EndpointSlice resources.
- `k3s-upgrade/` and `system-upgrade-controller/` manage K3s upgrades through Flux.
- The repository intentionally stays small: it describes routing endpoints, not application workloads.

## Routing

The `proxy-system` namespace contains a selectorless `proxy-service` plus manually managed EndpointSlices.

Current endpoint groups:

- `proxy-web-endpoint-slice`: HTTP/HTTPS traffic to `10.132.246.2` on ports `80` and `443`.
- `proxy-postgres-endpoint-slice`: PostgreSQL traffic to `10.132.246.2` on port `5432`.
- `proxy-smtp-endpoint-slice`: mail protocols to `10.132.246.4` on ports `25`, `465`, `587`, `143`, `993`, `110`, `995`, and `4190`.

Because the service has no selector, endpoint IPs are source-of-truth data in Git. Update the EndpointSlice manifests when backend private IPs change.

## Network Bootstrap

### Tailscale

```bash
sudo tailscale up \
  --accept-dns=false \
  --accept-routes \
  --reset
```

### Firewall

```bash
# Restrict SSH and the Kubernetes API to Tailscale.
sudo ufw deny 22/tcp
sudo ufw deny 6443/tcp
sudo ufw allow in on tailscale0 to any port 22 proto tcp
sudo ufw allow in on tailscale0 to any port 6443 proto tcp

# Public web ingress.
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Public mail protocols.
sudo ufw allow 25/tcp
sudo ufw allow 587/tcp
sudo ufw allow 465/tcp
sudo ufw allow 143/tcp
sudo ufw allow 993/tcp
sudo ufw allow 110/tcp
sudo ufw allow 995/tcp
sudo ufw allow 4190/tcp

# K3s cluster and service CIDRs.
sudo ufw allow from 10.63.0.0/16 to any
sudo ufw allow from 10.64.0.0/16 to any
```

## Bootstrap

### K3s

```bash
curl -fL https://get.k3s.io | \
sh -s - server \
  --disable traefik \
  --cluster-cidr=10.63.0.0/16 \
  --service-cidr=10.64.0.0/16 \
  --tls-san=100.110.44.59 \
  --tls-san=homelab-proxy-k3s.time-inconnu.ts.net \
  --tls-san=proxy.winetree94.com
```

### Flux

```bash
flux bootstrap github \
  --repository=proxy \
  --branch=main \
  --path=./clusters \
  --owner=tinyrack-net
```

## Operations

Render the Flux workload layer before pushing changes:

```bash
kubectl kustomize ./proxy
```

After Flux reconciliation, verify the generated service endpoints:

```bash
kubectl --context homelab-proxy -n proxy-system get service,endpointslice
```
