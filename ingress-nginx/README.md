# ingress-nginx

Disable Traefik in Rancher Desktop in Preferences > Kubernetes by unchecking Enable Traefik.

Install ingress-nginx using helm chart:
```bash
# update repo
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm repo list

# create namespace
kubectl create ns ingress-nginx
kubectl get ns

# install helm chart
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx

# check helm chart
helm list -A
helm get manifest ingress-nginx -n ingress-nginx
helm get values ingress-nginx -n ingress-nginx
helm get notes ingress-nginx -n ingress-nginx
```

### Setup certificates:
```bash
# Generate a Certificate Authority Certificate
# - Generate a CA certificate private key
openssl genrsa -out ca.key 4096
# - Generate the CA certificate
openssl req -x509 -new -nodes -sha512 -days 365 -subj "/CN=MyPersonal Root CA SDCK" -key ca.key -out ca.crt

# add certificate to store
certutil -addstore root ..\ca.crt
```