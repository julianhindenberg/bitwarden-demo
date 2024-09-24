```shell
# Install K3s
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -s - \
  --disable=traefik

# Create directories for persistent volumes
mkdir -p /data/Bitwarden/volume{1,2,3,4,5,6,7}
# Adjust permissions for MSSQL volumes
chown 10001:10001 /data/Bitwarden/volume{5,6,7}

# Install ingress nginx
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.externalTrafficPolicy=Local

# Create namespace
kubectl create namespace bitwarden

# Apply additional manifests
kubectl apply -f secrets.yaml -f persistent-volumes.yaml

# Install Bitwarden
helm upgrade bitwarden bitwarden/self-host --install --namespace bitwarden --values my-values.yaml

# (Configure domain override in /etc/hosts file)

# Request to IP endpoint
curl --insecure https://bitwarden.example.com/api/ip
```