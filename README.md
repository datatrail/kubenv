# Local setup ISE

## Rancher

# Setup Rancher using Powershell (admin mode)

```ps
# uninstall rancher
choco uninstall rancher-desktop

# delete rancher-desktop and rancher-desktop-data if exists
wsl --list
wsl --unregister rancher-desktop
wsl --unregister rancher-desktop-data

# delete dir appdata
Remove-Item "$env:USERPROFILE\AppData\Local\rancher-desktop" -Recurse -Force

# (re)install rancher
choco install rancher-desktop
```

## Ingress-nginx

Install [ingress-nginx](ingress-nginx).


## Argocd (todo)


## Harbor
Install [harbor](harbor).



## Kestra