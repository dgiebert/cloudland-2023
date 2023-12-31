---
weight: 4
title: "Install the Elemental Operator"
---

## Install the Elemental Operator
K3s includes a Helm Controller that manages installing, upgrading/reconfiguring, and uninstalling Helm charts using a HelmChart Custom Resource Definition (CRD). Paired with auto-deploying AddOn manifests, installing a Helm chart on your cluster can be automated by creating a single file on disk.

1. Create `/var/lib/rancher/k3s/server/manifests/` with the following content:
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: cattle-elemental-system
    ---
    apiVersion: helm.cattle.io/v1
    kind: HelmChart
    metadata:
      name: elemental-operator-crds
      namespace: cattle-elemental-system
    spec:
      chart: oci://registry.opensuse.org/isv/rancher/elemental/dev/charts/rancher/elemental-operator-crds-chart
      version: 1.2.4
    ---
    apiVersion: helm.cattle.io/v1
    kind: HelmChart
    metadata:
      name: elemental-operator
      namespace: cattle-elemental-system
    spec:
      chart: oci://registry.opensuse.org/isv/rancher/elemental/dev/charts/rancher/elemental-operator-chart
      version: 1.2.5
    ```

### Sources
- https://docs.k3s.io/helm
- https://elemental.docs.rancher.com/elementaloperatorchart-reference