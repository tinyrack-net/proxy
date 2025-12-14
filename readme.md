```bash
curl -fL https://get.k3s.io | \
sh -s - server \
  --cluster-cidr=10.63.0.0/16 \
  --service-cidr=10.64.0.0/16
```

```bash
flux bootstrap gitea \
  --token-auth \
  --hostname=git.winetree94.com \
  --repository=proxy \
  --branch=main \
  --path=./clusters/production \
  --owner=tinyrack
```
