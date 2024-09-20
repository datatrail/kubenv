# Harbor

[Configure HTTPS Access to Harbor](https://goharbor.io/docs/2.0.0/install-config/configure-https/)

### Setup certificates:
```bash
# Generate a Server Certificate
# - Generate a private key.
openssl genrsa -out harbor.sdck.nl.key 4096
# - Generate a certificate signing request (CSR)
openssl req -sha512 -new -subj "/CN=harbor.sdck.nl" -key harbor.sdck.nl.key -out harbor.sdck.nl.csr
# - Generate an x509 v3 extension file
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=harbor.sdck.nl
EOF
# - Use the v3.ext file to generate a certificate for your Harbor host
openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ..\ca.crt -CAkey ..\ca.key -CAcreateserial -in harbor.sdck.nl.csr -out harbor.sdck.nl.crt
```

Install harbor:

Create namespace:
```bash
kubectl create ns harbor
```

Add tls secret:
```bash
kubectl create secret tls harbor-tls --cert=harbor.sdck.nl.crt --key=harbor.sdck.nl.key -n harbor
```

Get values.yaml from Helm:
```bash
helm repo add harbor https://helm.goharbor.io
helm search repo harbor
helm repo update
helm repo list
helm show values harbor/harbor > values.yaml
```

Update ingress in values.yaml:
```bash
expose.tls.certSource: secret
expose.tls.secret.secretName: harbor-tls
expose.ingress.className: nginx
expose.ingress.hosts.core: harbor.sdck.nl
externalURL: https://harbor.sdck.nl
```

Convert to Kustomize and install argocd:
```bash
helm template harbor harbor/harbor -f values.yaml --output-dir harbor-manifests -n harbor

cd .\harbor-manifests\harbor\templates\
kustomize create --autodetect --recursive

kubectl apply -n harbor -k .
```