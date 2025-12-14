```bash
# SSH 포트 허용
sudo ufw allow ssh

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
sudo ufw allow 6443/tcp

# K3S 클러스터 및 서비스 CIDR 허용
sudo ufw allow from 10.63.0.0/16 to any
sudo ufw allow from 10.64.0.0/16 to any
```

```bash
curl -fL https://get.k3s.io | \
sh -s - server \
  --disable traefik \
  --cluster-cidr=10.63.0.0/16 \
  --service-cidr=10.64.0.0/16
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
