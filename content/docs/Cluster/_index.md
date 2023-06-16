---
title: Create the Cluster
type: docs
weight: 3
---

# Create from existing nodes 

# UI
1. Click the three dots and then select Create Elemental Cluster
    ![Create Cluster](/create-cluster.png)

2. Provided the needed details
    ![Cluster Settings 1](/cluster-settings-1.png)
    ![Cluster Settings 2](/cluster-settings-2.png)

# Using HelmChart

```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: elemental-cluster-1
  namespace: fleet-default
spec:
  chart: oci://ghcr.io/dgiebert/cluster-template-examples
  version: 0.0.3
  valuesContent: |-
    cloudprovider: elemental
    kubernetesVersion: "v1.24.13+k3s1"
    rke:
      machineGlobalConfig:
        secrets-encryption: true
      machineSelectorConfig:
      - config:
          docker: false
          protect-kernel-defaults: true
          selinux: false
    nodepools:
    - name: elemental-server
      etcd: true
      controlplane: true
      worker: true
      matchLabels:
        registrationEndpoint: rpi
```