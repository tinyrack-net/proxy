```bash
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 25/tcp
sudo ufw allow 6443/tcp
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
