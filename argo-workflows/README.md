# Argo Workflows

### Setup certificates:
```bash
# Generate a Server Certificate
# - Generate a private key
openssl genrsa -out argowf.sdck.nl.key 4096
# - Generate a certificate signing request (CSR)
openssl req -sha512 -new -subj "/CN=argowf.sdck.nl" -key argowf.sdck.nl.key -out argowf.sdck.nl.csr
# - Generate an x509 v3 extension file
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=argowf.sdck.nl
EOF
# - Use the v3.ext file to generate a certificate for your Harbor host
openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ..\ca.crt -CAkey ..\ca.key -CAcreateserial -in argowf.sdck.nl.csr -out argowf.sdck.nl.crt
```

### Install argo workflows (https://www.youtube.com/watch?v=UgpVX5Sn5OI)

Create namespace: 
```bash
kubectl create ns argowf
```

Add tls secret:
```bash
kubectl create secret tls argowf-tls --cert=argowf.sdck.nl.crt --key=argowf.sdck.nl.key -n argowf
```

Get values.yaml from Helm:
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm search repo argo
helm repo update
helm repo list
```

Update values.yaml:
```bash
server:
  extraArgs:
    - --auth-mode=server
  ingress: 
    enabled: true
    ingressClassName: nginx
    host:
      - argowf.sdck.nl
```

Install Helm chart:
```
helm install argowf argo/argo-workflows -n argowf -f values.yaml
```