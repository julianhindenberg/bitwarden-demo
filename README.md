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

# Encode and insert the installation id and installation key to secrets.yaml
echo "my-installation-id" | base64

# Apply additional manifests
kubectl apply -f secrets.yaml -f persistent-volumes.yaml

# Install Bitwarden
helm repo add bitwarden https://charts.bitwarden.com/
helm repo update
helm upgrade bitwarden bitwarden/self-host --install --namespace bitwarden --values my-values.yaml

# (Configure domain override in /etc/hosts file for IPv4 and IPv6)

# Perform calls to Bitwarden, either via
curl --insecure -6 https://bitwarden.example.com/
# or via browser.

# Check the API pod logs. Entries like the following will occur when connecting via IPv6:
DeviceType: Origin: ClientVersion:
      Unknown proxy: [::ffff:10.42.0.28]:41432
```