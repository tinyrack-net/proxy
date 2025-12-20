```bash
sudo tailscale up \
  --accept-dns=false \
  --accept-routes \
  --reset
```

```bash
# Tailscale Network 에서의 SSH 와 Kubernetes API 포트 허용
sudo ufw deny 22/tcp
sudo ufw deny 6443/tcp
sudo ufw allow in on tailscale0 to any port 22 proto tcp
sudo ufw allow in on tailscale0 to any port 6443 proto tcp

# HTTP/HTTPS 포트 허용
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Mail 관련 포트 허용
sudo ufw allow 25/tcp
sudo ufw allow 587/tcp
sudo ufw allow 465/tcp
sudo ufw allow 143/tcp
sudo ufw allow 993/tcp
sudo ufw allow 110/tcp
sudo ufw allow 995/tcp
sudo ufw allow 4190/tcp

# K3S 클러스터 및 서비스 CIDR 허용
sudo ufw allow from 10.63.0.0/16 to any
sudo ufw allow from 10.64.0.0/16 to any
```

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

```bash
flux bootstrap gitea \
  --token-auth \
  --hostname=git.winetree94.com \
  --repository=proxy \
  --branch=main \
  --path=./clusters \
  --owner=tinyrack
```
