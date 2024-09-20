# ArgoCD

### Setup certificates:
```bash
# Generate a Server Certificate
# - Generate a private key
openssl genrsa -out argocd.sdck.nl.key 4096
# - Generate a certificate signing request (CSR)
openssl req -sha512 -new -subj "/CN=argocd.sdck.nl" -key argocd.sdck.nl.key -out argocd.sdck.nl.csr
# - Generate an x509 v3 extension file
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=argocd.sdck.nl
EOF
# - Use the v3.ext file to generate a certificate for your Harbor host
openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ..\ca.crt -CAkey ..\ca.key -CAcreateserial -in argocd.sdck.nl.csr -out argocd.sdck.nl.crt
```

### Install argocd (https://devopscube.com/setup-argo-cd-using-helm/)

Create namespace: 
```bash
kubectl create ns argocd
```

Add tls secret:
```bash
kubectl create secret tls argocd-tls --cert=argocd.sdck.nl.crt --key=argocd.sdck.nl.key -n argocd
```

Get values.yaml from Helm:
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm search repo argo
helm repo update
helm repo list
helm show values argo/argo-cd > values.yaml
```

Update ingress in values.yaml:
```bash
global.domain: argocd.sdck.nl
server.ingress.enables: true
server.ingress.annotations:
      nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
      nginx.ingress.kubernetes.io/ssl-passthrough: "true"
      nginx.ingress.kubernetes.io/backend-protocol: "HTTPS" # handle too many redirects!!
server.ingress.ingressClassName: "nginx"
server.ingress.extraTls: 
      - hosts:
        - argocd.sdck.nl
        secretName: argocd-tls
```

Convert to Kustomize and install argocd:
```bash
helm template argo argo/argo-cd -f values.yaml --output-dir argocd-manifests -n argocd

cd .\argocd-manifests\argo-cd\templates\
kustomize create --autodetect --recursive

kubectl apply -n argocd -k .
```