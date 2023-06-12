---
weight: 2
title: "Installing Rancher"
bookToc: false
---

# Installing Rancher Managment Server

1. [Install Cert Manager](https://ranchermanager.docs.rancher.com/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster#4-install-cert-manager)
    ```sh
    helm repo add jetstack https://charts.jetstack.io
    helm repo update
    helm install cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --create-namespace \
    --version v1.11.0 \
    --set installCRDs=true
    ```
2. [Install Rancher](https://ranchermanager.docs.rancher.com/pages-for-subheaders/install-upgrade-on-a-kubernetes-cluster#5-install-rancher-with-helm-and-your-chosen-certificate-option)
    ```sh
    helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
    helm repo update
    helm install rancher rancher-latest/rancher \
    --namespace cattle-system \
    --create-namespace \
    --set hostname=cloudland.giebert.dev \
    --set global.cattle.psp.enabled=false \
    --set ingress.tls.source=letsEncrypt \
    --set letsEncrypt.email=me@giebert.dev \
    --set replicas=1
    ```
3. Wait for the Rollout
    ```sh
    kubectl -n cattle-system rollout status deploy/rancher
    ```
5. Navigate to https://cloudland.giebert.dev

### Sources
- https://github.com/rancher/rancher/
- https://ranchermanager.docs.rancher.com